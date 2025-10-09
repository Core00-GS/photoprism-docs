# Label Generation

PhotoPrism can enrich your library with AI-generated labels in addition to its [built-in classifiers](classification.md). This guide shows how to connect PhotoPrism directly to an [Ollama](https://ollama.com/search?c=vision) instance so that a multimodal large language model (LLM) can produce structured label metadata alongside the default TensorFlow models.


!!! tldr ""
    This walkthrough covers a **direct Ollama API integration**. Power users who prefer a fully managed proxy and Python extensibility should look at the dedicated [Vision Service](service/index.md) instead.

!!! warning ""
    The Ollama integration is **under active development**. Behaviour, configuration keys, and CLI commands may still change. Please report regressions so we can keep this documentation current.

## Ollama Setup Guide

Follow these steps to install Ollama, download a multimodal model, and configure PhotoPrism to request labels through the Ollama API.

### Step 1: Install Ollama

Add an `ollama` service next to PhotoPrism in your `compose.yaml` (or `docker-compose.yml`). Most templates published on our [download server](https://dl.photoprism.app/docker/compose.yaml) already contain this section and can be started with:

```bash
docker compose --profile ollama up -d
```

!!! example "compose.yaml"
    ```yaml
    services:
      photoprism:
        image: photoprism/photoprism:preview
        # ...

      ollama:
        image: ollama/ollama:latest
        restart: unless-stopped
        stop_grace_period: 15s
        environment:
          OLLAMA_HOST: "0.0.0.0:11434"
          OLLAMA_MODELS: "/root/.ollama"
          OLLAMA_MAX_QUEUE: "100"
          OLLAMA_NUM_PARALLEL: "1"
          OLLAMA_MAX_LOADED_MODELS: "1"
          OLLAMA_LOAD_TIMEOUT: "5m"
          OLLAMA_KEEP_ALIVE: "5m"
          OLLAMA_CONTEXT_LENGTH: "4096"
          OLLAMA_MULTIUSER_CACHE: "false"
          OLLAMA_NOPRUNE: "false"
          OLLAMA_NOHISTORY: "true"
          OLLAMA_FLASH_ATTENTION: "false"
          OLLAMA_KV_CACHE_TYPE: "f16"
          OLLAMA_SCHED_SPREAD: "false"
          OLLAMA_NEW_ENGINE: "true"
          # OLLAMA_DEBUG: "true"
        volumes:
          - "./ollama:/root/.ollama"
        # Enable NVIDIA acceleration only on trusted hosts:
        # deploy:
        #   resources:
        #     reservations:
        #       devices:
        #         - driver: "nvidia"
        #           capabilities: [ gpu ]
        #           count: "all"
    ```

Run Ollama **only** inside trusted networks. The service currently has no built-in authentication.

### Step 2: Download Models
Then pull at least one multimodal model that can output structured JSON, for example:

```bash
docker compose exec ollama ollama pull gemma3:latest
```

Other viable options already validated in our tests include qwen2.5vl:3b, qwen2.5vl:3b-q8_0, and qwen2.5vl:3b-fp16. Stick to lightweight quantizations when running on CPUs and reserve FP16 variants for GPUs.

### Step 3: Configure PhotoPrism

Create or update the `config/vision.yml` file in the PhotoPrism storage directory (inside the container: `/photoprism/storage/config/vision.yml`). Reuse the built-in defaults and append an Ollama-backed labels model:

!!! example "vision.yml"
    ```yaml
    Models:
    - Type: nsfw
      Default: true
    - Type: face
      Default: true
    - Type: labels
      Name: gemma3
      Version: latest
      Engine: ollama
      Run: newly-indexed # Run after indexing completes to avoid slowing imports
      Prompt: |
        You are a vision model that returns JSON labels describing the main subjects,
        objects, setting, and activities. Provide 3 to 6 concise nouns or noun phrases.
        Respond in English and keep values short.
      Options:
        Temperature: 0.1
      Service:
        Uri: "http://ollama:11434/api/generate"
    Thresholds:
      Confidence: 10
    ```

The `Run` field controls when PhotoPrism should execute this model (see [Scheduling label runs](#scheduling-label-runs)). Setting `Run: newly-indexed` defers evaluation to the metadata worker, preventing long-running LLM calls from slowing down imports.

!!! note ""
    The configuration file **must** be named `vision.yml` (not `vision.yaml`). Invalid indentation or spelling causes PhotoPrism to ignore the model and fall back to its default TensorFlow classifier.

### Step 4: Restart PhotoPrism

Apply the new configuration by restarting the app container:

```bash
docker compose stop photoprism
docker compose up -d
```

## Scheduling Label Runs

Each entry in `vision.yml` may define a `Run: <mode>` property so administrators can decide when PhotoPrism executes a model. Supported values map to the contexts below (the value is case-insensitive):

| Run value       | Description                                                                                           |
|-----------------|-------------------------------------------------------------------------------------------------------|
| `manual`        | Only runs when you invoke the model explicitly via CLI or API (`photoprism vision run`).             |
| `on-index`      | Runs in-line with the main indexing workflow. Best suited to fast, local models to avoid slow imports.|
| `newly-indexed` | Defers execution until after indexing finishes so slower models can run via the metadata worker.      |


## Troubleshooting

### Verify Active Configuration

```bash
docker compose exec photoprism photoprism vision ls
```

Check that the output lists both the default Nasnet classifier and your Ollama labels model. Missing entries usually indicate indentation issues, typos in the file name, or conflicting environment overrides.

### Schema or JSON Errors

If PhotoPrism logs `vision: invalid label payload from ollama`, confirm that:

- `Format: json` is set on the model definition.
- The prompt tells the LLM to honour the schema and stay in one language (append "Respond in German" when localising).
- The schema is valid JSON. Validate external schema files with `jq . vision-schema.json`.

PhotoPrism automatically falls back to Nasnet whenever it cannot parse the custom response.

### Latency and Timeouts

Structured responses cause slightly higher latency. Increase the service timeout (for example, `ServiceTimeout` in advanced deployments) if requests time out, and keep `Options.Temperature` low to encourage deterministic output.

### GPU Considerations

Long-running Ollama sessions on GPUs can suffer from VRAM fragmentation. Restart the Ollama container to recover performance:

```bash
docker compose down ollama
docker compose up -d ollama
```

### Tested Models

- `gemma3:latest` returns broad industrial labels in roughly 17 seconds and respects the JSON schema.
- `qwen2.5vl:3b` and its quantised variants followed the default prompt in our tests and produced reliable label arrays.

Experiment with prompt wording gradually and leave the TensorFlow fallback enabled so metadata stays consistent if the model emits empty or mixed-language responses.

