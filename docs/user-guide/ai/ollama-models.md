# Ollama Models

We recommend choosing a [vision model](https://ollama.com/search?c=vision) that balances speed, accuracy, and performance. Two models that meet these criteria and that we can recommend are [Gemma 3](https://ollama.com/library/gemma3) and [Qwen3-VL](https://ollama.com/library/qwen3-vl):

| Model             | Use Case                               | Notes                                                                |
|-------------------|----------------------------------------|----------------------------------------------------------------------|
| `gemma3:latest`   | Standard captions and labels           | Light, reliable JSON output; good default.                           |
| `qwen3-vl:latest` | Advanced vision (OCR, complex prompts) | Better visual grounding and multi-language support; needs more VRAM. |

[**Gemma 3**](https://ollama.com/library/gemma3) is very consistent in terms of performance and has very few errors/glitches. However, it is less likely to perform well with long/complex prompts and captions.

[**Qwen3-VL**](https://ollama.com/library/qwen3-vl) can be less predictable, especially in the `2b` and `4b` variants, where performance and error rates vary widely. The recommended `qwen3-vl:latest` (`8b`) version has performed well thus far, though we haven't tested it extensively. We know, however, that the previous version, Qwen2.5-VL, was good at generating long captions in different languages and styles. The trade-off is slightly lower performance compared to Gemma 3 on an NVIDIA RTX 4060: 2–3 seconds vs 1–2 seconds for generating labels.

Performance also depends on your hardware, so e.g., Qwen3-VL might outperform Gemma 3 when running on Apple Silicon or NVIDIA Blackwell GPUs. Our recommendation is therefore to test both models to see which one works best for you. If you generate both captions and labels, stick with this model so that Ollama doesn't need to swap models between requests.

!!! tldr ""
    Without GPU acceleration, Ollama models will be significantly slower, taking anywhere from 10 seconds to over a minute to complete. This may be acceptable if you only want to process a few pictures or are willing to wait.

## Configuration Examples

When the `Engine` is set to `ollama`, PhotoPrism applies suitable defaults (720 px thumbnails, JSON prompts for labels). You may therefore omit the `Prompt` and `Options` shown in the examples below if you would like to use the defaults.

### Gemma 3: Adjusting the Temperature & Top P for Labels

```yaml
Models:
  - Type: labels
    Model: gemma3:latest
    Engine: ollama
    Run: newly-indexed
    Options:
      Temperature: 0.01
      TopP: 0.95
    Service:
      Uri: http://ollama:11434/api/generate
```

### Qwen3-VL: Specifying a Caption Prompt

```yaml
Models:
  - Type: caption
    Model: qwen3-vl:latest
    Engine: ollama
    Run: on-schedule
    Prompt: |
      Write one or two sentences in natural language
      that describe the main subject, actions, and setting.
      Avoid meta-language and formatting.
    Options:
      Temperature: 0.05
    Service:
      Uri: http://ollama:11434/api/generate
```

You can use the following prompt to generate concise captions with exactly one sentence:

> Create a caption with exactly one sentence in the active voice that describes the main visual content. Begin with the main subject and clear action. Avoid text formatting, meta-language, and filler words.

Example: *A sleek pool extends over a dramatic cliffside overlooking turquoise waters.*

As an alternative, try this prompt to generate detailed captions of up to three sentences:

> Write a descriptive caption in 3 sentences or fewer that captures the essence of the visual content. Avoid text formatting, meta-language, and filler words. Do not start captions with phrases such as "This image", "The picture", or "Here are". Begin with the subject(s), then describe the surroundings, and finally add atmosphere (e.g., time of day). If possible, include the subject's gender and general age group.

Example: *A gray cat with a fluffy coat is lounging on a cushion, its eyes closed in a peaceful slumber. The background features a blurred view of trees and a blue sky, suggesting it's daytime. The cat's relaxed posture and the serene outdoor setting create a tranquil and cozy atmosphere.*

## Usage Tips

- Use `Run: newly-indexed` for everyday workloads; switch to `on-schedule` or `manual` for larger, slower models you want to control.
- If responses come back empty with [Qwen3-VL](https://ollama.com/library/qwen3-vl), upgrade to [PhotoPrism 251130](https://github.com/photoprism/photoprism/releases/tag/251130-b3068414c) or later so the adapter reads the `thinking` field Ollama returns.
- When tuning prompts, keep them short and include a schema reminder for labels. Overly long prompts can increase hallucinations and latency.
