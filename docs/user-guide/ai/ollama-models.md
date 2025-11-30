# Ollama Models

Choose a model that balances speed, accuracy, and hardware needs. PhotoPrism applies safe defaults (720 px thumbnails, JSON prompts for labels) whenever `Engine: ollama` is set.

## Recommendations

| Model             | Use Case                               | Notes                                                                                           |
|-------------------|----------------------------------------|-------------------------------------------------------------------------------------------------|
| `gemma3:latest`   | Default for captions and labels        | Light, reliable JSON output; works well as a general default. Expect ≥10× slower on CPU.        |
| `qwen3-vl:latest` | Advanced vision (OCR, complex prompts) | Better visual grounding and multi-language support; needs more VRAM. Expect ≥10× slower on CPU. |

!!! tldr ""
    On an NVIDIA RTX 4060, generating labels for an image with Ollama usually takes 1-3 seconds. Note that without GPU acceleration, it will be 10 times slower or more.

## Configuration

### Gemma 3

```yaml
Models:
  - Type: labels
    Name: gemma3:latest
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
    Name: qwen3-vl:latest
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
