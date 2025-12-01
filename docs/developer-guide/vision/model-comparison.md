# Vision Model Comparison

## Caption Generation

We tested the ability of the following [Ollama Vision models](https://ollama.com/search?c=vision) to generate accurate, natural-sounding captions for 100 different images, including photos, drawings, and memes:

* **Qwen2.5-VL (3B & 7B)**
* **Moondream**
* **MiniCPM-V**
* **Llama3.2-Vision**
* **Granite3.2-Vision**
* **Gemma 3**

The results were evaluated based on the following **quality criteria**:

1. Formatting (avoid Markdown, meta-language, and filler words)
2. Text recognition (OCR)
3. Number of subjects, including their gender and age, if applicable
4. Location description including Landmark recognition, if applicable
5. Date and/or time estimation based on visible information
6. Image type classification (e.g. illustration, drawing, watercolor)

!!! tldr ""
    To ensure comparability, all tests were run on an **AMD Ryzen AI 9 365 CPU**, and three different prompts were used for each model. A GPU will significantly reduce the time needed per image.

### General Observations

* Larger models benefit from **longer prompts**, but too much instruction can increase hallucination rates.
* If a model hallucinates on >50% of images, try **shortening the prompt** (e.g., Gemma 3).
* All tested models struggled with:
    - Captioning **grids of multiple images** (often skipping some images).
    - **Non-English or partially obscured text**.
    - OCR accuracy was highest for **clear English text**.

### Qwen2.5-VL — **Overall Winner**

[Qwen2.5-VL](https://ollama.com/library/qwen2.5vl), developed by Alibaba, provided the **most accurate and consistent captions** across nearly all criteria:

* Strong in **OCR, subject age, gender, location, landmark, and image type recognition**.
* Captions were **natural, detailed, and well-structured**.
* Rarely (<1%) inserted Chinese characters.
* Always generated **medium-to-long captions** (3+ sentences) — does not follow requests to shorten.

#### Performance

* **Average time per image:** 36.34s (7B) / 24.12s (3B)
* **Hallucination rate:** 0.33% (7B) / 1.33% (3B)
* **Error rate:** 18% (7B) / 19% (3B)

#### Compatibility

Qwen2.5-VL [requires Ollama 0.7.0](https://ollama.com/library/qwen2.5vl#readme) and may not be compatible with [later versions](https://github.com/ollama/ollama/releases). If the regular version doesn't work, try the FP16 variant as a workaround, e.g. `qwen2.5vl:3b-fp16`.

#### Prompts

We recommend the following prompt to generate concise captions with exactly one sentence:

> Create a caption with exactly one sentence in the active voice that describes the main visual content. Begin with the main subject and clear action. Avoid text formatting, meta-language, and filler words.

Example: *A sleek pool extends over a dramatic cliffside overlooking turquoise waters.*

As an alternative, this prompt will generate detailed captions of up to three sentences:

> Write a descriptive caption in 3 sentences or fewer that captures the essence of the visual content. Avoid text formatting, meta-language, and filler words. Do not start captions with phrases such as "This image", "The picture", or "Here are". Begin with the subject(s), then describe the surroundings, and finally add atmosphere (e.g., time of day). If possible, include the subject's gender and general age group.

Example: *A gray cat with a fluffy coat is lounging on a cushion, its eyes closed in a peaceful slumber. The background features a blurred view of trees and a blue sky, suggesting it's daytime. The cat's relaxed posture and the serene outdoor setting create a tranquil and cozy atmosphere.*

#### Summary

Use `qwen2.5vl:7b` for **detailed captions** with **high accuracy**. For better performance on lower-spec systems, we recommend using `qwen2.5vl:3b` instead, which provides slightly lower detail and accuracy. To generate captions in other languages, keep the prompt in English and specify the desired language.

### Gemma 3 — **Best Lightweight Model**

[Gemma 3](https://ollama.com/library/gemma3), a Google Gemini-based model, offers **fast and lightweight captioning** with reasonable accuracy:

* Excels at **single-sentence captions**.
* Good **OCR for large, clear text**.
* Performs best with **short prompts** (<500 characters).
* More prone to hallucination with longer prompts.

#### Performance

* **Average time per image:** 31.02s
* **Hallucination rate:** 2%
* **Error rate:** 25%

#### Compatibility

[Gemma 3](https://ollama.com/library/gemma3) should be compatible with the [latest Ollama versions](https://github.com/ollama/ollama/releases). Please [report](https://github.com/photoprism/photoprism-docs/tree/develop/docs/developer-guide/vision/model-comparison.md) any issues you experience so that we can include them in our documentation.

#### Prompt

We recommend using the following prompt to generate short, concise captions:

> Create a caption with exactly one sentence in the active voice that describes the main visual content.
Begin with the main subject and clear action. Avoid text formatting, meta-language, and filler words.

Example: *A snow-covered church stands prominently on a winter street.*

#### Summary

If you prioritize **speed** and **simplicity** over detailed captions, the `gemma3:4b` / `gemma3:latest` model is a great choice. To generate captions in other languages, keep the prompt in English and specify the desired language.

### Key Takeaways

* **Qwen2.5-VL:7B** → Best overall accuracy, detailed multi-sentence captions, slower.
* **Qwen2.5-VL:3B** → Nearly as good, faster, less strong at OCR.
* **Gemma 3:4B** → Lightweight, single-sentence captions, best for quick results.
* Other models, including Moondream, MiniCPM-V, Llama3.2-Vision, and Granite3.2-Vision, performed poorly in at least one quality criterion and had higher hallucination and error rates.

!!! example ""
    We welcome contributions to our computer vision documentation. If you have any additions or suggestions for improvements, please click the :material-file-edit-outline: button in the upper right corner of the page to send a pull request.