# Ollama Models

We recommend choosing a [vision model](https://ollama.com/search?c=vision) that balances speed, accuracy, and performance. 
Two models that meet these criteria and that we can recommend are [Gemma 3](https://ollama.com/library/gemma3) and [Qwen3-VL](https://ollama.com/library/qwen3-vl):

| Model             | Use Case                               | Notes                                                                |
|-------------------|----------------------------------------|----------------------------------------------------------------------|
| `gemma3:latest`   | Standard captions and labels           | Light, reliable JSON output; good default.                           |
| `qwen3-vl:latest` | Advanced vision (OCR, complex prompts) | Better visual grounding and multi-language support; needs more VRAM. |

!!! tldr ""
    On an NVIDIA RTX 4060, generating labels for an image with Ollama usually takes 1-3 seconds. Without GPU acceleration, however, it can be significantly slower, taking anywhere from 10 seconds to over a minute.

## Configuration

PhotoPrism applies suitable defaults (720 px thumbnails, JSON prompts for labels) when the engine is set to Ollama, so prompts and options may be omitted.

### Gemma 3

```yaml
Models:
  - Type: labels
    Model: gemma3:latest
    Engine: ollama
    Run: newly-indexed
    Options:
      Temperature: 0.1
      TopP: 0.9
    Service:
      Uri: http://ollama:11434/api/generate
```

Prompt Example:

> Analyze the image and return a JSON object with `labels`, each containing `name`, `confidence` (0-1), and `topicality` (0-1). Keep names as single nouns.

### Qwen3-VL

```yaml
Models:
  - Type: caption
    Model: qwen3-vl:latest
    Engine: ollama
    Run: on-schedule
    Options:
      Temperature: 0.05
    Service:
      Uri: http://ollama:11434/api/generate
```

Prompt Example:

> Write one or two sentences in natural language that describe the main subject, actions, and setting. Avoid meta-language and formatting.

## Usage Tips

- Use `Run: newly-indexed` for everyday workloads; switch to `on-schedule` or `manual` for larger, slower models you want to control.
- If responses come back empty with Qwen3-VL, upgrade to PhotoPrism 2025.11 or later so the adapter reads the `thinking` field Ollama returns.
- When tuning prompts, keep them short and include a schema reminder for labels. Overly long prompts can increase hallucinations and latency.
