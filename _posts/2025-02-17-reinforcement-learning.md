---
title: "Reinforcement Learning for Molecular Generation"
date: 2025-02-16 22:00:00 -0400
categories: [Tech, Machine Learning]
tags: [machine-learning, reinforcement-learning]
image: /assets/blog_files/2025-02-17-reinforcement-learning/RL-concept.excalidraw.png
math: true
---

## 1 Introduction 

> Reinforcement learning (RL) is an interdisciplinary area of machine learning and optimal control concerned with how an intelligent agent should take actions in a dynamic environment in order to maximize a reward signal.  
> Ref: [Wiki of RL](https://en.wikipedia.org/wiki/Reinforcement_learning)  

Resources of RL ([Recommendedy by reddit user](https://www.reddit.com/r/reinforcementlearning/comments/embtnp/comment/fdnl09t/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)):   
1. [Spinning Up in RL](https://spinningup.openai.com/en/latest/)! - created by OpenAI to serve as introduction to modern Deep RL, I would start here.
https://spinningup.openai.com/en/latest/spinningup/rl_intro.html
2. [Reinforcement Learning: An Introduction by Sutton and Barto](https://book.douban.com/subject/30323890/) - is the introductory book on RL. It covers very basics and build up from there. Does not cover deep RL. Another link: incompleteideas.net/book/the-book-2nd.html.
3. [David Silver Reinforcement Learning Course (2016)](https://www.youtube.com/playlist?list=PLbWDNovNB5mqFBgq7i3MY6Ui4zudcvNFJ) - video lectures aligned with book above. Very well explained, covers basics and some modern deep rl methods. Very fun to watch IMHO.
4. [Advanced Deep Learning & Reinforcement Learning (2018)](https://www.youtube.com/watch?v=iOh7QUZGyiU&list=PLqYmG7hTraZDNJre23vqCGIVpfZ_K2RZs) - updated version of the above, more slower paced, but some things are better explained in 2016 version IMHO.
5. [Deep Reinforcement Learning](https://arxiv.org/abs/1810.06339) - 2018 paper by Yuxi Li is a recent(ish) survey and overview of the field.



## 2 RL to generate molecule with highest MW

### 2.1 Problem statement

Let's play a game! The SMILES string must be exactly 3 characters long, and only single bonds are allowed. What is the highest molecular weight (MW) you can achieve? Use only C, N, and O to find the valid SMILES with the highest MW.  

### 2.2 Direct search

We can solve this problem simply by using brute-force search. Below are the code:   
```python
from rdkit import Chem
from rdkit.Chem import Descriptors
import itertools

# Possible characters for SMILES
atoms = ["C", "N", "O"]

# Generate all possible SMILES combinations
possible_smiles = set()
for comb in itertools.product(atoms, repeat=3):
    smiles = "".join(comb)
    if len(smiles) == 3:
        possible_smiles.add(smiles)
print("Check the combinations: \n")
print(possible_smiles)
print(len(possible_smiles))

# Validate SMILES and find highest MW
best_smiles = None
highest_mw = 0

for smi in possible_smiles:
    mol = Chem.MolFromSmiles(smi)
    if mol:  # Check if valid
        mw = Descriptors.MolWt(mol)
        if mw > highest_mw:
            highest_mw = mw
            best_smiles = smi

print(f"Best SMILES: {best_smiles}, Highest MW: {highest_mw}")
```

The computation complexity can be very expensive. When the atoms number is 3, the search space is 3^3. When the atoms number become 10, the search space is 10^10. Therefore, this method is not optimal.  

### 2.3 RL with Q-learning

However, we can use reinforcement learning to solve this problem. Below is the main component of reinforcement learning:  
![alt text](/assets/blog_files/2025-02-17-reinforcement-learning/RL-concept.excalidraw.png)  

We use two classes to define **agent** and **environment**. Below is the code:  
```python
import random
import pickle  # For saving and loading the model
from rdkit import Chem
from rdkit.Chem import Descriptors

# Allowed elements
elements = ["C", "N", "O"]
# SMILES length constraint
max_length = 3


# Molecular Environment
class MolecularEnvironment:
    def __init__(self):
        self.current_smiles = []
    
    def reset(self):
        """Resets the environment (clears SMILES string)."""
        self.current_smiles = []
        return ""

    def step(self, action):
        """Takes an action (adds an element) and returns new state, reward, and done flag."""
        self.current_smiles.append(elements[action])
        new_state = "".join(self.current_smiles)
        done = len(self.current_smiles) == max_length  # Stop when 3 characters are reached
        reward = self.get_molecular_weight(new_state) if done else 0
        return new_state, reward, done

    @staticmethod
    def get_molecular_weight(smiles):
        """Calculates molecular weight using RDKit. Returns -10 for invalid molecules."""
        mol = Chem.MolFromSmiles(smiles)
        return Descriptors.MolWt(mol) if mol else -10  # Penalize invalid SMILES


# Q-learning Agent
class Agent:
    def __init__(self, alpha=0.1, gamma=0.9, epsilon=0.2):
        self.q_table = {}  # State-action values
        self.alpha = alpha  # Learning rate (how much new information influences past values)
        self.gamma = gamma  # Discount factor (importance of future rewards)
        self.epsilon = epsilon  # Exploration rate

def train_molgen_highMW(agent, episodes=10):
    """Trains the agent using Q-learning to maximize molecular weight."""
    env = MolecularEnvironment()

    for episode in range(episodes):
        state = env.reset()  # Start with an empty SMILES
        done = False
        
        while not done and len(state) < max_length:
            # Choose action (explore or exploit)
            if random.random() < agent.epsilon:
                action = random.choice(range(len(elements)))  # Random exploration
            else:
                action = max(range(len(elements)), key=lambda a: agent.q_table.get((state, a), 0))  # Exploitation
                
            new_state, reward, done = env.step(action)

            # Q-learning update
            old_value = agent.q_table.get((state, action), 0)
            next_max = max([agent.q_table.get((new_state, a), 0) for a in range(len(elements))], default=0)
            # standard Bellman equation for Q-learning
            agent.q_table[(state, action)] = old_value + agent.alpha * (reward + agent.gamma * next_max - old_value)

            state = new_state  # Move to new state


# Start training
agent = Agent()
train_molgen_highMW(agent)

agent.q_table
```

Change the `episodes` number to see the changes of Q table.  

Some quick notes:
1. Bellman equation is used to update q-value:  
$$
Q(s, a) \leftarrow Q(s, a) + \alpha \left( r + \gamma \max_{a'} Q(s', a') - Q(s, a) \right)
$$
2. The same state-action pair can be updated multiple times as the agent learns.
3. Each update gradually moves Q-values toward the true expected return.
4. Q-values stabilize over time as learning progresses.
5. We can draw the q-table by ourselves. The wiki shows the blue print of the table (Refer to [Q-learning wiki](https://en.wikipedia.org/wiki/Q-learning)). For our example, we can draw a table like this (easy to read for human):   

| All State | Add C | Add O | Add N |
| --------- | ----- | ----- | ----- |
| `""`      | ?     | ?     | ?     |
| `"C"`     | ?     | ?     | ?     |
| `"CN"`    | ?     | ?     | ?     |


Here is the full code: [Code_RL for molecular generation](https://colab.research.google.com/github/BaosenZ/baosenz.github.io/blob/main/assets/blog_files/2025-02-17-reinforcement-learning/code_RLmolgen/RLmolgen_highMW.ipynb) [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/BaosenZ/baosenz.github.io/blob/main/assets/blog_files/2025-02-17-reinforcement-learning/code_RLmolgen/RLmolgen_highMW.ipynb).

## 3 Future plan

In the future, I want to make it more complex and generate molecules with drug-likeness. 

