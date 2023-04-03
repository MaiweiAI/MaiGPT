# MaiGPT

![image](https://user-images.githubusercontent.com/29084184/229440418-28a6dd9c-97a5-43f0-b595-b42f65b3e4c5.png)


## Overview

Language serves as an interface for LLMs to connect numerous AI models for solving complicated AI tasks!

<p align="center"><img src="https://user-images.githubusercontent.com/29084184/229440526-16b6593c-19b9-4fe0-af99-e7d7b49ffaa5.png"></p>

We introduce a collaborative system that consists of **an LLM as the controller** and **numerous expert models as collaborative executors** (from HuggingFace Hub). The workflow of our system consists of four stages:
+ **Task Planning**: Using ChatGPT to analyze the requests of users to understand their intention, and disassemble them into possible solvable sub-tasks.
+ **Model Selection**: Based on the sub-tasks, ChatGPT invoke the corresponding models hosted on HuggingFace.
+ **Task Execution**: Executing each invoked model and returning the results to ChatGPT.
+ **Response Generation**: Finally, using ChatGPT to integrate the prediction of all models, and generate response.

## System Requirements

+ Ubuntu 16.04 LTS
+ NVIDIA GeForce RTX 3090 * 1
+ RAM > 24GB

## Quick Start

First replace `openai.key` and `huggingface.cookie` in `server/config.yaml` with **your personal key** and **your cookies at huggingface.co**. Then run the following commands:

### For server:

```bash
# setup env
git clone https://github.com/microsoft/JARVIS
cd server
conda create -n jarvis python=3.8
conda activate jarvis
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r requirements.txt

# download models
cd models
sh download.sh

# run server
cd ..
python models_server.py
python bot_server.py --config config.yaml # for text-davinci-003
```

### For web:

```bash
cd web
npm install
npm run dev
```
Note that in order to display the video properly in HTML, you need to compile `ffmpeg` manually with H.264

```bash
# This command need be executed without errors.
LD_LIBRARY_PATH=/usr/local/lib /usr/local/bin/ffmpeg -i input.mp4 -vcodec libx264 output.mp4
```

### For CLI:

```bash
cd server
python awesome_chat.py
```

Example of CLI mode running:

```bash
System: Welcome to Jarvis! A collaborative system that consists of an LLM as the controller and numerous expert models as collaborative executors. Please enter your request (`exit` to exit).
Input: Given a collection of image A: /examples/cat.jpg, B: /examples/z3.jpg, C: /examples/z1.jpg, please tell me how many zebras in these picture?
Output: Based on your request, I have used several models to process the images you provided. For the image A: /examples/cat.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "a cat sitting on a window sill looking out". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/a3dc.jpg', which contains two objects with labels 'potted plant' and 'cat'. For the image B: /examples/z3.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "two zebras standing in a field of dry grass". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/f5df.jpg', which contains three objects with labels 'zebra'. For the image C: /examples/z1.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "a zebra standing in a field of tall grass". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/69c7.jpg', which contains one object with label 'zebra'. Finally, I used the canny-control model canny-control to generate the image '/images/7e18.png' based on the image /examples/savanna.jpg. Therefore, there are four zebras in these pictures. Is there anything else I can help you with?
Input: Please answer all the named entities in the sentence: Iron Man is a superhero appearing in American comic books published by Marvel Comics. The character was co-created by writer and editor Stan Lee, developed by scripter Larry Lieber, and designed by artists Don Heck and Jack Kirby.
Output: Sure, I understand your request. Based on the inference results of the models, I will answer your request step by step. Firstly, I used the token-classification model dslim/bert-base-NER to recognize the named entities in the sentence. The inference result is that the named entities are Iron Man, Stan Lee, Larry Lieber, Don Heck and Jack Kirby. Then, I used the question-answering model deepset/roberta-base-squad2 to answer your request. The inference result is that there is no output since the context cannot be empty. Therefore, I cannot make it. I hope this answer your request. Is there anything else I can help you with?
Input: Please generate a canny image based on /examples/savanna.jpg
Output: Sure. I have generated a canny image based on /examples/savanna.jpg. To do this, I first used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text description of the image, which is "a herd of giraffes and zebras grazing in a field". Then I used the canny-control model to generate the canny image of the original image, which is stored in the path "/images/1ea2.png". Finally, I used the canny-text-to-image model lllyasviel/sd-controlnet-canny to generate the canny image based on the text description and the canny image, which is stored in the path "/images/ba66.png". Do you need anything else?
```


## Configuration

The server-side configuration file is `server/config.yaml`, and some parameters are presented as follows:

+ `model`: LLM, currently supports `text-davinci-003`
+ `inference_mode`: mode of inference endpoints
  + `local`: only use the local inference endpoints
  + `huggingface`: use the Hugging Face Inference Endpoints and local ControlNet Endpoints
  + `hybrid`: both of `local` and `huggingface`
+ `local_models`: scale of locally deployed models:
  +  `minimal` (RAM>24GB, ControlNet only)
  +  `standard` (RAM>40GB, ControlNet + Standard Pipelines)
  +  `full` (RAM>80GB, All registered models)

On the personal laptop, we recommend the configuration of `inference_mode: huggingface `and `local_models: minimal`. However, due to the instability of remote Hugging Face Inference Endpoints, the services provided by expert models may be limited.

## Screenshots

<p align="center"><img src="https://user-images.githubusercontent.com/29084184/229441159-915a5c20-39a3-467c-ba5a-38b2c4f50cf1.png"><img src="https://user-images.githubusercontent.com/29084184/229440796-55ae7db2-d2e2-484f-96e6-e0e574961720.png"></p>

## Acknowledgement

- [HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in HuggingFace](http://arxiv.org/abs/2303.17580)
- [ChatGPT](https://platform.openai.com/)
- [HuggingFace](https://huggingface.co/)
- [ControlNet](https://github.com/lllyasviel/ControlNet)
- [ChatGPT-vue](https://github.com/lianginx/chatgpt-vue)
