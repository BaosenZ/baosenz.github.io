---
title: "Finetune Llama and Deploy Web Chatbot"
date: 2025-02-24 21:00:00 -0400
categories: [Tech, LLMs]
tags: [machine-learning, large-language-models, web-development]
image: /assets/blog_files/2025-02-24-finetune-Llama-and-deploy-web-chatbot/profile.excalidraw.png
math: true
---

## 1 Unsloth Finetune 

Based on the [documentation of Unsloth](https://docs.unsloth.ai/), we can do the finetune with Google Colab. They provide us the [Colab notebook code](https://docs.unsloth.ai/get-started/unsloth-notebooks). We can just copy it our Colab to run them. I personally ran the code in NERSC computer. It is really faster than in free version Colab! It lets me know the power of becoming rich~     

I played with a [Mental health dataset](https://huggingface.co/datasets/Amod/mental_health_counseling_conversations). A post in hf (Refer to [Fine-Tuning 1B LLaMA 3.2: A Comprehensive Step-by-Step Guide with Code](https://huggingface.co/blog/ImranzamanML/fine-tuning-1b-llama-32-a-comprehensive-article)) provide the guide. Below are some key points that I learned:   
1. Unsloth inference includes the input, but Ollama run does not.  
Detailed explanation: After fine-tuning with Unsloth, the inference process automatically includes the input. The generated response comes after our input, so an additional step is needed to extract the response. However, when deploying the model with Ollama, its output does not include the input—it directly provides the response.
2. What is the input of a transformer?  
It is actually the entire text! Detailed explanation: During training, both the input and output are provided to the model. In our training data, this means the entire text column is used. During inference, the model predicts the next word based on the given input.
3. What we are using is essentially instruction tuning.  
The only difference is that we have fixed the instruction to focus on mental health.
4. What is our input?  
We provide the model with the entire prompt format, including the response content. The entire text column is given to the model.
5. What are the benefits of this approach?  
During inference, we include the input format as well. When the LLM sees a similar instruction, it tends to follow the patterns learned from our training data and generates a response accordingly. This ensures that the generated response is similar to our training data. Therefore, when developing an application, we need to include the prompt format as part of the entire input.
6. A model I trained on Colab often includes the format in its output.
However, a model trained on NERSC and then run using Ollama from Hugging Face rarely includes the format. This issue seems to be related to echoing the input format, which is undesirable. I believe the model trained on NERSC performs better!
7. During training, I feel using wandb to visualize the training process is convenient. The [wandb documentation about this integration](https://docs.wandb.ai/guides/integrations/huggingface/) gave us instructions how to use. The main code of this integration is just below:  

```python
os.environ["WANDB_PROJECT"] = "<my-amazing-project>"  # name your W&B project
os.environ["WANDB_LOG_MODEL"] = "checkpoint"  # log all model checkpoints
from transformers import TrainingArguments, Trainer

args = TrainingArguments(..., report_to="wandb")  # turn on W&B logging
trainer = Trainer(..., args=args)
```

8. After finishing traing, we can directly run the code below to save the model to hf:   

```python
# Save to 8bit Q8_0
# Remember to go to https://huggingface.co/settings/tokens for a token!
# And change <hf> to your username!
if True: model.push_to_hub_gguf("<hf>/model", tokenizer, token = "")
```

## 2 Run our model locally

We will use ollama to run our model locally. I wrote a guide in [this post](https://baosenz.github.io/posts/run-deepseekr1-local-windows/). For our situation, we can just run 
```bash
ollama run https://huggingface.co/bzhang143/mental-health-model
```
This is very simple! Just copy the link in hf. I don't need to download the model! The code above will pull the model and run in your machine.   
Use `ollama list` to find our model name, whic should be "huggingface.co/bzhang143/mental-health-model". When we write our app, remember to change the model in the `client.chat()`. 

## 3 Web development

We can make a Flask app. The image below shows the web app:  
![alt text](/assets/blog_files/2025-02-24-finetune-Llama-and-deploy-web-chatbot/image.png){: w="500" h="300" }
_Deploy Finetuned Llama locally and make a web app._

Here is some quick notes:
1. When we use ollama to create client, pass the host name. If it is a localhost, we write `host="http://localhost:11434"`. Or we can just ignore. If we want to create a docker compose, like one host ollama server, another one host Flask app, we can pass the host name to other like `host="http://localhost:11434"`. At this situation, you cannot ignore.  
2. Notice my `text_prompt`! Accually, the inputs to the Llama is not the user typed message, it is the `text_prompt`+ user typed message! When I finetuen the llama model, we provide this template. This is a intruction tuning.  
3. Change the model in the `client.chat()`. 
4. In our html template, we use `var socket = io.connect(window.location.origin);` to automatically detect host. In our local PC, it should detect `http://localhost:5000`.  
5. In the future, learn more about [WebSocket](https://en.wikipedia.org/wiki/WebSocket). 
6. Maybe, in the future, look at this [Open WebUI](https://github.com/open-webui/open-webui) to let this chatbot looking better while simply implemented.   

Below is the full code.   
```python
from flask import Flask, render_template
from flask_socketio import SocketIO, emit
import ollama

app = Flask(__name__)
socketio = SocketIO(app, cors_allowed_origins="*")  # Enable WebSockets

client = ollama.Client(host="http://localhost:11434")
# client = ollama.Client(host="http://llmserve:11434")


@app.route("/")
def index():
    return render_template("index.html")


@socketio.on("user_message")
def handle_user_message(data):
    user_input = data["message"]
    # print(f"User: {user_input}")

    text_prompt = f"""Analyze the provided text from a mental health perspective. Identify any indicators of emotional distress, coping mechanisms, or psychological well-being. Highlight any potential concerns or positive aspects related to mental health, and provide a brief explanation for each observation.

    ### Input:
    {user_input}

    ### Response:
    """
    # Force model to complete after "Response:"

    # Query the Ollama model
    response = client.chat(
        model="huggingface.co/bzhang143/mental-health-model",
        messages=[{"role": "user", "content": text_prompt}],
    )
    bot_response = response["message"]["content"]
    print(f"Bot: {bot_response}")

    extracted_bot_response = bot_response.split("### Response:", 1)[-1]

    emit("bot_response", {"message": extracted_bot_response})


if __name__ == "__main__":
    socketio.run(app, host="0.0.0.0", port=5000, debug=True)
```

Create `templates` folder and `index.html` file, write the code below:  
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ollama Chatbot</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.4.1/socket.io.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }

        .chat-box {
            width: 50%;
            margin: auto;
            padding: 10px;
            border: 1px solid #ccc;
            height: 400px;
            overflow-y: auto;
        }

        .input-box {
            width: 50%;
            margin: auto;
            display: flex;
        }

        input {
            flex: 1;
            padding: 10px;
        }

        button {
            padding: 10px;
            cursor: pointer;
        }
    </style>
</head>

<body>
    <h2 style="text-align:center">Ollama Chatbot: Mental Health</h2>
    <div class="chat-box" id="chat-box"></div>
    <div class="input-box">
        <input type="text" id="user-input" placeholder="Type a message..." />
        <button onclick="sendMessage()">Send</button>
    </div>

    <script>
        // var socket = io.connect("http://localhost:5000");
        var socket = io.connect(window.location.origin);  // Automatically detect host

        function sendMessage() {
            var inputField = document.getElementById("user-input");
            var message = inputField.value.trim();
            if (message === "") return;

            addMessage("You: " + message, "user");
            socket.emit("user_message", { message: message });

            inputField.value = "";
        }

        function addMessage(text, sender) {
            var chatBox = document.getElementById("chat-box");
            var messageDiv = document.createElement("div");
            messageDiv.textContent = text;
            chatBox.appendChild(messageDiv);
            chatBox.scrollTop = chatBox.scrollHeight;
        }

        socket.on("bot_response", function (data) {
            addMessage("Bot: " + data.message, "bot");
        });

        document.getElementById("user-input").addEventListener("keypress", function (event) {
            if (event.key === "Enter") {
                sendMessage();
            }
        });
    </script>
</body>

</html>
```