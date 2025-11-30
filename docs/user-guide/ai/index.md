# Introduction

In addition to the built-in TensorFlow models, PhotoPrism allows you to generate captions and labels through direct Ollama and OpenAI API integrations. The following guides explain how to set up Ollama, create a custom `vision.yml` configuration, choose the AI engines and vision models to use, and customize the prompts according to your needs.

## Supported Engines

PhotoPrism currently supports the following AI frameworks and services:

| Engine                        | Resolution | Runs        | Best For                                                                       |                      
|-------------------------------|------------|-------------|--------------------------------------------------------------------------------|
| TensorFlow (default)          | 224 px     | Built-in    | Offline labels, NSFW, Facial Recognition                                       | 
| [Ollama](using-ollama.md)     | 720 px     | Self-Hosted | Good for generating quality captions & labels, but a GPU is recommended        | 
| [OpenAI API](using-openai.md) | 720 px     | Cloud       | Provides high-quality captions & labels, also suitable for users without a GPU | 

!!! tldr ""
    On an NVIDIA RTX 4060, generating labels for an image with Ollama usually takes 1-3 seconds. Note that without GPU acceleration, it will be 10 times slower or more.

## Setup and Usage Guides

- [Using Ollama](using-ollama.md)
- [Ollama Models](ollama-models.md)
- [Using the OpenAI Responses API](using-openai.md)
- [Face Recognition](face-recognition.md)
