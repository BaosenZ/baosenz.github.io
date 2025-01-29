---
title: "Run DeepSeek-R1 in Our Local Windows Computer"
date: 2025-01-25 22:00:00 -0400
categories: [Tech, LLMs]
tags: [tech, machine-learning, large-language-models]
image: /assets/blog_files/2025-01-23-run-deepseekr1-local-windows/image-1.png
---

## 1 Use DeepSeek-R1 with Ollma (Quick start)

1. We can install Ollama, and here is the hyperlink, [Ollama](https://ollama.com/). We can install Ollama in Windows. The procedure is straightforward.  

2. Once installation is done, we can run our first model by typing the command in terminal,
```bash
ollama run llama3.2:1b
```
This will pull the llama3.2 model with 1b params. The full model library is avaiable in ollama, here is the hyperlink, [llama model library](https://ollama.com/search). After the pulling is done, it will run the model in the terminal. Below is the image:    
![alt text](/assets/blog_files/2025-01-23-run-deepseekr1-local-windows/image-1.png){: w="500" h="300" }
_ollama run llama3.2:1b model._

2. The Ollama has already intergrated deepseek-r1 model. So, we can quickly pull and run the deepseek-r1 model with ollma locally, and we can check all downloaded models. Below is the code, 
```bash
ollama run deepseek-r1:1.5b
ollama list  # show all llm models
```
Image belowshows what I ran in my computer:  
![alt text](/assets/blog_files/2025-01-23-run-deepseekr1-local-windows/image-3.png){: w="500" h="300" }
_ollama run deepseek-r1:1.5b model._

Hardware requirements: [To Do]   
llama model requirement: https://llamaimodel.com/requirements-3-2

## 2 Create Our Model using Ollama

We can create our model with source code DeepSeek team published in HuggingFace. You can find the link in the GitHub, and here is the hyperlink, [GitHub repo: DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1.git).  

### 2.1 Download DeepSeek model

Make sure we have git lfs installed, by typing, 
```bash
git lfs --version
```
If not, follow the guides to install git lfs first. 

git clone deepseek-r1 model locally. 
```bash
cd <to/the/path/we/want/to/store/model>
git clone https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B
```
This will take several minutes.   

### 2.2 Create our model

The ollama documentation provides many information about creating our model, here are twp useful hyperlink, [import a model](https://github.com/ollama/ollama/blob/main/docs/import.md) and [modelfile](https://github.com/ollama/ollama/blob/main/docs/modelfile.md). Below is the steps. 

1. Create `Modelfile` in a Windows folder, write inside this file
```
FROM ../DeepSeek-R1-Distill-Qwen-1.5B
```
>Note that just the folder name in `Modelfile` should be fine!
{: .prompt-info }

2. Now run the `ollama create` command from the directory where you created the `Modelfile`:
```bash
ollama create my-deepseekr1-1.5b
```
Lastly, test the model:
```bash
ollama run my-deepseekr1-1.5b
```
Image below shows the process:   
![alt text](/assets/blog_files/2025-01-23-run-deepseekr1-local-windows/image-4.png){: w="500" h="300" }
_create deepseek-r1 model using ollama with source._


## 3 Write Python

We can use "ollama" Python package to call our newly created model, below is the code: 
```python
import ollama

# Initialize the Ollama client
client = ollama.Client()

# Define the model and the input prompt
model = "my-deepseekr1-1.5b"  # Replace with your model name
prompt = "Hello, introduce yourself with 50 words!"

# Send the query to the model
response = client.generate(model=model, prompt=prompt)

# Print the response from the model
print("Response from Ollama:")
print(response.response)
```

A more tedious way is to requests to call our model (not recommended), below is the code, 
```python
import requests
import json

# Set up the base URL for the local Ollama API
# We can find the port by view logs in Windows ollam tray icon
url = "http://localhost:11434/api/chat"

# input prompt
payload = {
    "model": "my-deepseekr1-1.5b",  # Replace with the model name you're using
    "messages": [{"role": "user", "content": "Hello, introduce yourself with 50 words!"}]
}

# Send the HTTP POST request with streaming enabled
response = requests.post(url, json=payload, stream=True)

# Check the response status
if response.status_code == 200:
    print("Streaming response from Ollama:")
    for line in response.iter_lines(decode_unicode=True):
        if line:  # Ignore empty lines
            try:
                # Parse each line as a JSON object
                json_data = json.loads(line)
                # Extract and print the assistant's message content
                if "message" in json_data and "content" in json_data["message"]:
                    print(json_data["message"]["content"], end="")
            except json.JSONDecodeError:
                print(f"\nFailed to parse line: {line}")
    print()  # Ensure the final output ends with a newline
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

## Reference

1. DeepSeek-R1: [https://github.com/deepseek-ai/DeepSeek-R1.git](https://github.com/deepseek-ai/DeepSeek-R1.git)
2. Ollama: [https://ollama.com/](https://ollama.com/); Ollama documentation: [https://github.com/ollama/ollama/tree/main/docs](https://github.com/ollama/ollama/tree/main/docs), Ollama model library: [https://ollama.com/search](https://ollama.com/search).
3. The Youtube video talk other models but a great to learn Ollama, and code for playing with API is from this video. Here is the link, [https://www.youtube.com/watch?v=UtSSMs6ObqY&t=687s&ab_channel=TechWithTim](https://www.youtube.com/watch?v=UtSSMs6ObqY&t=687s&ab_channel=TechWithTim)
4. Another video, a good thing is we can get web service. Here is the link, [https://www.youtube.com/watch?v=TpNwYA8Eqhk&ab_channel=CodingCrashCourses](https://www.youtube.com/watch?v=TpNwYA8Eqhk&ab_channel=CodingCrashCourses)