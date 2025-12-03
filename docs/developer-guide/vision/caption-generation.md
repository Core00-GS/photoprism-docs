# Caption Generation

As an addition to its [built-in AI capabilities](tensorflow/index.md), PhotoPrism lets you generate image captions through a direct [Ollama](https://ollama.com/search?c=vision) integration, as [described in this guide](#ollama-setup-guide).

It allows you to choose from the [available vision models](https://ollama.com/search?c=vision) and [customize the prompts](../../user-guide/ai/ollama-models.md#gemma-3-caption) according to your needs.

## Ollama Setup Guide

Follow the [steps in our User Guide](../../user-guide/ai/using-ollama.md) to connect PhotoPrism directly to an Ollama instance and generate captions with [vision-capable LLMs](https://ollama.com/search?c=vision).

[Learn more ›](../../user-guide/ai/using-ollama.md)

## Troubleshooting ##

### Verifying Your Configuration ###

If you encounter issues, a good first step is to verify how PhotoPrism has loaded your `vision.yml` configuration. You can do this by running: 

```bash
docker compose exec photoprism photoprism vision ls
```

This command outputs the settings for all supported and configured model types. Compare the results with your `vision.yml` file to confirm that your configuration has been loaded correctly and to identify any parsing errors or misconfigurations.

### GPU Performance Issues ###

When using Ollama with GPU acceleration, you may experience performance degradation over time due to VRAM management issues. This typically manifests as processing times gradually increasing and the Ollama service appearing to "crash" while still responding to requests, but without GPU acceleration.

The issue occurs because Ollama's VRAM allocation doesn't properly recover after processing multiple requests, leading to memory fragmentation and eventual GPU processing failures.

The Ollama service does not automatically recover from these VRAM issues. To restore full GPU acceleration, manually restart the Ollama container:

```bash
docker compose down ollama
docker compose up -d ollama
```

This will clear the VRAM and restore normal GPU-accelerated processing performance.
