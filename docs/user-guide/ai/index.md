# Using AI Models

As an addition to the built-in TensorFlow models, PhotoPrism lets you generate captions and labels with [Ollama](using-ollama.md) and the [OpenAI API](using-openai.md). Our step-by-step guides explain how to set them up and provide tested configuration examples you can use as a starting point.

[Learn more ›](using-ollama.md)

## Model Engines

PhotoPrism currently supports the following runtimes and services:

| Engine                                                                 | Resolution | Runs        | Best For                                                                                                      |                      
|------------------------------------------------------------------------|------------|-------------|---------------------------------------------------------------------------------------------------------------|
| [TensorFlow](../../developer-guide/vision/tensorflow/custom-models.md) | 224 px     | Built-in    | Fast, offline default models for core features (labels, faces, NSFW)                                          | 
| [Ollama](using-ollama.md)                                              | 720 px     | Self-Hosted | Good for generating quality captions & labels; a server with GPU is recommended                               | 
| [OpenAI API](using-openai.md)                                          | 720 px     | Cloud       | Highest quality captions & labels, also suitable for users without a GPU; requires API key and network access | 

### Performance

- **TensorFlow:** Our built-in models generally perform well on all types of hardware.
- **Ollama:** On an NVIDIA RTX 4060, generating labels for an image with Ollama usually takes 1-3 seconds. Without GPU acceleration, however, it can be significantly slower, taking anywhere from 10 seconds to over a minute.
- **OpenAI:** Processing one image takes about 3 seconds, though this can vary by model, region, and demand.

## `vision.yml` Reference

Custom AI engines, models, and run modes can be specified in a `vision.yml` file located in the `storage/config` directory. The file defines a list of models and thresholds to be used, e.g.:

```yaml
Models:
- Type: caption
  Model: gemma3:latest
  Engine: ollama
  Run: newly-indexed
  Options:
    Temperature: 0.05
  Service:
    Uri: http://ollama:11434/api/generate
- Type: labels
  Model: qwen3-vl:latest
  Engine: ollama
  Service:
    Uri: http://ollama:11434/api/generate
Thresholds:
  Confidence: 10
  Topicality: 0
  NSFW: 75
```

If a model type is omitted, PhotoPrism will use the built-in defaults for `labels`, `nsfw`, `face`, or `caption`. The optional `Thresholds` block can be used to filter out labels with a low probability or adjust the probability of flagging content as NSFW. 

| Field                          | Default                                | Notes                                                                              |
|--------------------------------|----------------------------------------|------------------------------------------------------------------------------------|
| `Type` (required)              | —                                      | `labels`, `caption`, `face`, `nsfw`. Drives routing & scheduling.                  |
| `Model`                        | `""`                                   | Raw identifier override; precedence: `Service.Model` → `Model` → `Name`.           |
| `Name`                         | derived from type/version              | Display name; lower-cased by helpers.                                              |
| `Version`                      | `latest` (non-OpenAI)                  | OpenAI payloads omit version.                                                      |
| `Engine`                       | inferred from service/alias            | Aliases set formats, file scheme, resolution. Explicit `Service` values still win. |
| `Run`                          | `auto`                                 | See Run modes table below.                                                         |
| `Default`                      | `false`                                | Keep one per type for TensorFlow fallbacks.                                        |
| `Disabled`                     | `false`                                | Registered but inactive.                                                           |
| `Resolution`                   | 224 (TensorFlow) / 720 (Ollama/OpenAI) | Thumbnail edge in px; TensorFlow models default to 224 unless you override.        |
| `System` / `Prompt`            | engine defaults / empty                | Override prompts per model.                                                        |
| `Format`                       | `""`                                   | Response hint (`json`, `text`, `markdown`).                                        |
| `Schema` / `SchemaFile`        | engine defaults / empty                | Inline vs file JSON schema (labels).                                               |
| `TensorFlow`                   | engine defaults / empty                | Local TF model info (paths, tags).                                                 |
| [`Options`](#model-options)    | engine defaults / empty                | Sampling/settings merged with engine defaults.                                     |
| [`Service`](#service-settings) | engine defaults / empty                | Remote endpoint config (see below).                                                |

### Run Modes

| Value           | When it runs                                                     | Recommended use                                |
|-----------------|------------------------------------------------------------------|------------------------------------------------|
| `auto`          | TensorFlow defaults during index; external via metadata/schedule | Leave as-is for most setups.                   |
| `manual`        | Only when explicitly invoked (CLI/API)                           | Experiments and diagnostics.                   |
| `on-index`      | During indexing + manual                                         | Fast built-in models only.                     |
| `newly-indexed` | Metadata worker after indexing + manual                          | External/Ollama/OpenAI without slowing import. |
| `on-demand`     | Manual, metadata worker, and scheduled jobs                      | Broad coverage without index path.             |
| `on-schedule`   | Scheduled jobs + manual                                          | Nightly/cron-style runs.                       |
| `always`        | Indexing, metadata, scheduled, manual                            | High-priority models; watch resource use.      |
| `never`         | Never executes                                                   | Keep definition without running it.            |

> For performance reasons, `on-index` is only supported for the built-in TensorFlow models.

### Options

The model `Options` allow you to adjust model parameters such as temperature, top-p, and schema constraints when using [Ollama](using-ollama.md)/[OpenAI](using-openai.md):

| Option            | Default                                                                                 | Description                                                                        |
|-------------------|-----------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| `Temperature`     | engine default (`0.1` for Ollama; unset for OpenAI)                                     | Controls randomness; clamped to `[0,2]`. `gpt-5*` OpenAI models are forced to `0`. |
| `TopP`            | engine default (`0.9` for some Ollama label defaults; unset for OpenAI)                 | Nucleus sampling parameter.                                                        |
| `MaxOutputTokens` | engine default (OpenAI caption 512, labels 1024; Ollama label default 256)              | Upper bound on generated tokens; adapters raise low values to defaults.            |
| `ForceJson`       | engine-specific (`true` for OpenAI labels; `false` for Ollama labels; captions `false`) | Forces structured output when enabled.                                             |
| `SchemaVersion`   | derived from schema name                                                                | Override when coordinating schema migrations.                                      |
| `Stop`            | engine default                                                                          | Array of stop sequences (e.g., `["\\n\\n"]`).                                      |
| `NumThread`       | runtime auto                                                                            | Caps CPU threads for local engines.                                                |
| `NumCtx`          | engine default                                                                          | Context window length (tokens).                                                    |

### Service

The model `Service` settings allow you to configure endpoints and authentication for engines that perform remote HTTP requests, such as [Ollama](using-ollama.md) and [OpenAI](using-openai.md):

| Field                              | Default                                  | Notes                                                |
|------------------------------------|------------------------------------------|------------------------------------------------------|
| `Uri`                              | required for remote                      | Endpoint base. Empty keeps model local (TensorFlow). |
| `Method`                           | `POST`                                   | Override verb if provider needs it.                  |
| `Key`                              | `""`                                     | Bearer token; prefer env expansion.                  |
| `Username` / `Password`            | `""`                                     | Injected as basic auth when URI lacks userinfo.      |
| `Model`                            | `""`                                     | Endpoint-specific override; wins over model/name.    |
| `Org` / `Project`                  | `""`                                     | OpenAI headers (org/proj IDs)                        |
| `RequestFormat` / `ResponseFormat` | set by engine alias                      | Explicit values win over alias defaults.             |
| `FileScheme`                       | set by engine alias (`data` or `base64`) | Controls image transport.                            |
| `Disabled`                         | `false`                                  | Disable the endpoint without removing the model.     |

> **Authentication:** All credentials and identifiers support `${ENV_VAR}` expansion. `Service.Key` sets `Authorization: Bearer <token>`; `Username`/`Password` injects HTTP basic authentication into the service URI when it is not already present.
