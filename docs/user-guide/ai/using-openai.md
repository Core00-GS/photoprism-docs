# Using the OpenAI Responses API

Learn how to configure PhotoPrism to use the OpenAI API for generating captions and labels.

The Responses API endpoint at `https://api.openai.com/v1/responses` is used for this, with a single 720 px thumbnail (`detail: low`). Engine defaults are applied when `Engine: openai` is set: base64 file scheme, JSON schema for labels, 512/1024 token limits, and deterministic sampling for GPT‑5 models (`temperature`/`top_p` forced to 0).

!!! tldr ""
    In order to use OpenAI services, you need a valid API key, which can be configured via `OPENAI_API_KEY` or `OPENAI_API_KEY_FILE`. PhotoPrism must also have network access to `api.openai.com`.

## Configuration

To use OpenAI, add the following caption and/or labels model configurations to your `vision.yml` file:

```yaml
Models:
  - Type: caption
    Name: gpt-5-nano
    Engine: openai
    Run: newly-indexed
    Options:
      Detail: low           # optional; default is low
      MaxOutputTokens: 512  # default applied automatically
    Service:
      Uri: https://api.openai.com/v1/responses
      Key: ${OPENAI_API_KEY}

  - Type: labels
    Name: gpt-5-mini
    Engine: openai
    Run: newly-indexed
    Options:
      Detail: low
      MaxOutputTokens: 1024
    Service:
      Uri: https://api.openai.com/v1/responses
      Key: ${OPENAI_API_KEY}
```

Recommendations:

- Keep the model `Name` exactly as published by OpenAI so defaults apply correctly.
- `Service.Key` can be omitted if `OPENAI_API_KEY` / `_FILE` is set in the environment.
- Optional headers: set `Service.Org` and `Service.Project` when your account requires them.

## Test Your Setup

1. Confirm models are registered:
   ```bash
   photoprism vision ls --json
   ```
2. Run a single photo to verify output:
   ```bash
   photoprism vision run -m caption --count 1 --force
   photoprism vision run -m labels --count 1 --force
   ```
3. If the response is unstructured, leave `ForceJson` enabled for labels; captions accept plain text.

## Usage Tips

- To avoid unexpected API costs, set `Run: manual` and run the models manually with the `photoprism vision run` command.
- When you remove custom model configuration from you `vision.yml` file, the built-in default models will be enabled again.
- When you need domain-specific wording, you may override `System` or `Prompt` in `vision.yml`; keep them short and retain the schema reminder for labels.
