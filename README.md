<img align="right" alt="KISS" src="https://user-images.githubusercontent.com/29084184/230841371-61f3b76b-d616-4921-922e-65b8647860cf.png" height="48" title="keep it simple, stupid">

# MaiGPT

<a src="https://img.shields.io/badge/%F0%9F%A4%97-Open%20in%20Spaces-blue" href="https://huggingface.co/spaces/microsoft/HuggingGPT">
    <img src="https://img.shields.io/badge/%F0%9F%A4%97-Open%20in%20Spaces-blue" alt="Open in Spaces">
</a>

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

### Default (Recommended)

For `config.yaml`:

+ Ubuntu 16.04 LTS
+ VRAM >= 24GB
+ RAM > 12GB (minimal), 16GB (standard), 80GB (full)
+ Disk > 284GB 
  + 42GB for `damo-vilab/text-to-video-ms-1.7b`
  + 126GB for `ControlNet`
  + 66GB for `stable-diffusion-v1-5`
  + 50GB for others
  
### Minimum (Lite)

For `lite.yaml`:

+ Ubuntu 16.04 LTS
+ Nothing else

The configuration `lite.yaml` does not require any expert models to be downloaded and deployed locally. However, it means that Jarvis is restricted to models running stably on HuggingFace Inference Endpoints.

## Quick Start

```
git clone https://github.com/microsoft/JARVIS
```

First replace `openai.key` and `huggingface.token` in `server/config.yaml` with **your personal OpenAI Key** and **your Hugging Face Token**. Then run the following commands:

<span id="Server"></span>

### For Server:

```bash
# setup env
cd server
conda create -n jarvis python=3.8
conda activate jarvis
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r requirements.txt

# download models. Make sure that `git-lfs` is installed.
cd models
bash download.sh # required when `inference_mode` is `local` or `hybrid`. 

# run server
cd ..
python models_server.py --config config.yaml # required when `inference_mode` is `local` or `hybrid`
python awesome_chat.py --config config.yaml --mode server # for text-davinci-003
```

Now you can access Jarvis' services by the Web API. 

+ `/hugginggpt` --method `POST`, access the full service.
+ `/tasks` --method `POST`, access intermediate results for Stage #1.
+ `/results` --method `POST`, access intermediate results for Stage #1-3.

For example:

```bash
# request
curl --location 'http://localhost:8004/tasks' \
--header 'Content-Type: application/json' \
--data '{
    "messages": [
        {
            "role": "user",
            "content": "based on pose of /examples/d.jpg and content of /examples/e.jpg, please show me a new image"
        }
    ]
}'

# response
[{"args":{"image":"/examples/d.jpg"},"dep":[-1],"id":0,"task":"openpose-control"},{"args":{"image":"/examples/e.jpg"},"dep":[-1],"id":1,"task":"image-to-text"},{"args":{"image":"<GENERATED>-0","text":"<GENERATED>-1"},"dep":[1,0],"id":2,"task":"openpose-text-to-image"}]
```


### For Web:

We provide a user-friendly web page. After starting `awesome_chat.py` in a server mode, you can run the commands to communicate with Jarvis in your browser:
 
- you need to install `nodejs` and `npm` first.
- [ IMPORTANT ] if you are running the web client on another machine, you need set `http://{LAN_IP_of_the_server}:{port}/` to `BASE_URL` of `web/src/config/index.ts`.
- if you want to use the video generation feature, you need to compile `ffmpeg` manually with H.264.
- you can switch to ChatGPT by `double click` on the setting icon!

```bash
cd web
npm install
npm run dev
```

```bash
# Optional: Install ffmpeg
# This command need be executed without errors.
LD_LIBRARY_PATH=/usr/local/lib /usr/local/bin/ffmpeg -i input.mp4 -vcodec libx264 output.mp4
```

<span id="Gradio"></span>

### For Gradio

The Gradio demo is now hosted on Hugging Face Space. You can also run the following commands to start the demo locally:

```bash
python models_server.py --config config.gradio.yaml
python run_gradio_demo.py --config config.gradio.yaml

# or run the HF Space as a Docker image (Build with `inference_mode=hibrid` and `local_deployment=standard`)
docker run -it -p 7860:7860 --platform=linux/amd64 registry.hf.space/microsoft-hugginggpt:latest python app.py
```

### For CLI:

You can also run Jarvis more easily in CLI mode:

```bash
cd server
python awesome_chat.py --config config.yaml --mode cli
```

Examples of CLI mode:

```
Welcome to Jarvis! A collaborative system that consists of an LLM as the controller and numerous expert models as collaborative executors. Jarvis can plan tasks, schedule Hugging Face models, generate friendly responses based on your requests, and help you with many things. Please enter your request (`exit` to exit).
[ User ]: Given a collection of image A: /examples/a.jpg, B: /examples/b.jpg, C: /examples/c.jpg, please tell me how many zebras in these picture?
[ Jarvis ]: Based on your request, I have used several models to process the images you provided. For the image A: /examples/a.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "a cat sitting on a window sill looking out". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/a3dc.jpg', which contains two objects with labels 'potted plant' and 'cat'. For the image B: /examples/b.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "two zebras standing in a field of dry grass". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/f5df.jpg', which contains three objects with labels 'zebra'. For the image C: /examples/c.jpg, I used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text "a zebra standing in a field of tall grass". Then I used the object-detection model facebook/detr-resnet-50 to generate the image with predicted box '/images/69c7.jpg', which contains one object with label 'zebra'. Finally, I used the canny-control model canny-control to generate the image '/images/7e18.png' based on the image /examples/f.jpg. Therefore, there are four zebras in these pictures. Is there anything else I can help you with?
[ User ]: Please answer all the named entities in the sentence: Iron Man is a superhero appearing in American comic books published by Marvel Comics. The character was co-created by writer and editor Stan Lee, developed by scripter Larry Lieber, and designed by artists Don Heck and Jack Kirby.
[ Jarvis ]: Sure, I understand your request. Based on the inference results of the models, I will answer your request step by step. Firstly, I used the token-classification model dslim/bert-base-NER to recognize the named entities in the sentence. The inference result is that the named entities are Iron Man, Stan Lee, Larry Lieber, Don Heck and Jack Kirby. Then, I used the question-answering model deepset/roberta-base-squad2 to answer your request. The inference result is that there is no output since the context cannot be empty. Therefore, I cannot make it. I hope this answer your request. Is there anything else I can help you with?
[ User ]: Please generate a canny image based on /examples/f.jpg
[ Jarvis ]: Sure. I have generated a canny image based on /examples/f.jpg. To do this, I first used the image-to-text model nlpconnect/vit-gpt2-image-captioning to generate the text description of the image, which is "a herd of giraffes and zebras grazing in a field". Then I used the canny-control model to generate the canny image of the original image, which is stored in the path "/images/1ea2.png". Finally, I used the canny-text-to-image model lllyasviel/sd-controlnet-canny to generate the canny image based on the text description and the canny image, which is stored in the path "/images/ba66.png". Do you need anything else?
```


## Configuration

The server-side configuration file is `server/config.yaml`, and some parameters are presented as follows:

+ `model`: LLM, currently supports `text-davinci-003`. We are working on integrating more open-source LLMs.
+ `inference_mode`: mode of inference endpoints
  + `local`: only use the local inference endpoints
  + `huggingface`: only use the Hugging Face Inference Endpoints **(free of local inference endpoints)**
  + `hybrid`: both of `local` and `huggingface`
+ `local_deployment`: scale of locally deployed models, works under `local` or `hybrid` inference mode:
  +  `minimal` (RAM>12GB, ControlNet only)
  +  `standard` (RAM>16GB, ControlNet + Standard Pipelines)
  +  `full` (RAM>42GB, All registered models)

On a personal laptop, we recommend the configuration of `inference_mode: hybrid `and `local_deployment: minimal`. But the available models under this setting may be limited due to the instability of remote Hugging Face Inference Endpoints.

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
