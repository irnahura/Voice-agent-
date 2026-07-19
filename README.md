<h1 align="center">VoiceAI</h1>
<p align="center">
  <p align="center"><b>End-to-end open-source voice agents platform</b>: Quickly build voice-first conversational assistants through a JSON configuration. </p>
</p>

<h4 align="center">
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="VoiceAI is released under the MIT license." />
  </a>
  <a href="#contributing">
    <img src="https://img.shields.io/badge/PRs-Welcome-brightgreen" alt="PRs welcome!" />
  </a>
</h4>

## Introduction

**VoiceAI** is an end-to-end open-source production-ready framework for quickly building LLM-based voice-driven conversational applications.

## What is this repository?
This repository contains the complete orchestration platform to build voice AI applications. It orchestrates voice conversations using combinations of different ASR (Automatic Speech Recognition) + LLM (Large Language Model) + TTS (Text-to-Speech) providers and models over WebSockets.


## Components
VoiceAI helps you create AI Voice Agents that can be instructed to perform various tasks through a simple configuration.

## Supported providers and models
1. Initiating a phone call using telephony providers like `Twilio`, `Plivo`, `Exotel` (coming soon), `Vonage` (coming soon) etc.
2. Transcribing the conversations using `Deepgram`, `Azure` etc.
3. Using LLMs like `OpenAI`, `DeepSeek`, `Llama`, `Cohere`, `Mistral`,  etc to handle conversations
4. Synthesizing LLM responses back to telephony using `AWS Polly`, `ElevenLabs`, `Deepgram`, `OpenAI`, `Azure`, `Cartesia`, `Smallest` etc.


See the provider configuration section below for detailed setup instructions.


## Local example setup
A basic local setup includes usage of [Twilio](local_setup/telephony_server/twilio_api_server.py) or [Plivo](local_setup/telephony_server/plivo_api_server.py) for telephony. The setup is dockerized in `local_setup/`. You'll need to populate an environment `.env` file from `.env.sample`.

The setup consists of four containers:

1. Telephony web server:
   * Choosing Twilio: You'll need to set up a [Twilio account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)
   * Choosing Plivo: You'll need to set up a [Plivo account](https://www.plivo.com/)
2. VoiceAI server: for creating and handling agents 
3. `ngrok`: for tunneling. You'll need to add the `authtoken` to `ngrok-config.yml`
4. `redis`: for persisting agent & prompt data

### Quick Start

The easiest way to get started is to use the provided script:

```bash
cd local_setup
chmod +x start.sh
./start.sh
```

This script will check for Docker dependencies, build all services with BuildKit enabled, and start them in detached mode.

### Manual Setup

Alternatively, you can manually build and run the services:

1. Make sure you have Docker with Docker Compose V2 installed
2. Enable BuildKit for faster builds:
   ```bash
   export DOCKER_BUILDKIT=1
   export COMPOSE_DOCKER_CLI_BUILD=1
   ```
3. Build the images:
   ```bash
   docker compose build
   ```
4. Run the services:
   ```bash
   docker compose up -d
   ```

To run specific services only:

```bash
docker compose up -d voiceai-app twilio-app
# or
docker compose up -d voiceai-app plivo-app
```

Once the docker containers are up, you can start creating your agents and initiating calls.

## Programmatic usage (minimal example)

You can also build and run an agent directly in Python without the local telephony setup.

Example script: `examples/simple_assistant.py`

```python
import asyncio
from voiceai.assistant import Assistant
from voiceai.models import (
    Transcriber,
    Synthesizer,
    ElevenLabsConfig,
    LlmAgent,
    SimpleLlmAgent,
)


async def main():
    assistant = Assistant(name="demo_agent")

    # Configure audio input (ASR)
    transcriber = Transcriber(provider="deepgram", model="nova-2", stream=True, language="en")

    # Configure LLM
    llm_agent = LlmAgent(
        agent_type="simple_llm_agent",
        agent_flow_type="streaming",
        llm_config=SimpleLlmAgent(
            provider="openai",
            model="gpt-4o-mini",
            temperature=0.3,
        ),
    )

    # Configure audio output (TTS)
    synthesizer = Synthesizer(
        provider="elevenlabs",
        provider_config=ElevenLabsConfig(
            voice="George", voice_id="JBFqnCBsd6RMkjVDRZzb", model="eleven_turbo_v2_5"
        ),
        stream=True,
        audio_format="wav",
    )

    # Build a single coherent pipeline: transcriber -> llm -> synthesizer
    assistant.add_task(
        task_type="conversation",
        llm_agent=llm_agent,
        transcriber=transcriber,
        synthesizer=synthesizer,
        enable_textual_input=False,
    )

    # Stream results
    async for chunk in assistant.execute():
        print(chunk)


if __name__ == "__main__":
    asyncio.run(main())
```

How to run:

```bash
export OPENAI_API_KEY=...
export DEEPGRAM_AUTH_TOKEN=...
export ELEVENLABS_API_KEY=...
python examples/simple_assistant.py
```

This demonstrates orchestration and streaming output. For telephony, use the services in `local_setup/`.

Note: For REST-based usage (Agent CRUD over HTTP), see `API.md` in the repo root.

Expected output shape: `assistant.execute()` is an async generator yielding per-task result dicts (event-like chunks). The exact keys depend on configured tools/providers; treat it as a stream and process incrementally.

### Text-only pipeline example

If you want a text-only flow (no transcriber/synthesizer), you can enable a text-only pipeline:

Example script: `examples/text_only_assistant.py`

```python
import asyncio
from voiceai.assistant import Assistant
from voiceai.models import LlmAgent, SimpleLlmAgent


async def main():
    assistant = Assistant(name="text_only_agent")

    llm_agent = LlmAgent(
        agent_type="simple_llm_agent",
        agent_flow_type="streaming",
        llm_config=SimpleLlmAgent(
            provider="openai",
            model="gpt-4o-mini",
            temperature=0.2,
        ),
    )

    # No transcriber/synthesizer; enable a text-only pipeline
    assistant.add_task(
        task_type="conversation",
        llm_agent=llm_agent,
        enable_textual_input=True,
    )

    async for chunk in assistant.execute():
        print(chunk)


if __name__ == "__main__":
    asyncio.run(main())
```

How to run (text-only):

```bash
export OPENAI_API_KEY=...
python examples/text_only_assistant.py
```

Expected output shape: `assistant.execute()` yields streaming dicts per task step; fields vary by configuration. Handle chunk-by-chunk.


## Using your own providers
You can populate the `.env` file to use your own keys for providers.

<details>

<summary>ASR Providers</summary><br>
These are the current supported ASRs Providers:

| Provider     | Environment variable to be added in `.env` file |
|--------------|-------------------------------------------------|
| Deepgram     | `DEEPGRAM_AUTH_TOKEN`                           |

</details>
&nbsp;<br>

<details>
<summary>LLM Providers</summary><br>
VoiceAI uses the LiteLLM package to support multiple LLM integrations.

These are the currently supported LLM Provider families:

For LiteLLM based LLMs, add either of the following to the `.env` file depending on your use-case:<br><br>
`LITELLM_MODEL_API_KEY`: API Key of the LLM<br>
`LITELLM_MODEL_API_BASE`: URL of the hosted LLM<br>
`LITELLM_MODEL_API_VERSION`: API VERSION for LLMs like Azure

For LLMs hosted via VLLM, add the following to the `.env` file:<br>
`VLLM_SERVER_BASE_URL`: URL of the hosted LLM using VLLM

</details>
&nbsp;<br>

<details>

<summary>TTS Providers</summary><br>
These are the currently supported TTS Providers:

| Provider   | Environment variable to be added in `.env` file  |
|------------|--------------------------------------------------|
| AWS Polly  | Accessed from system wide credentials via ~/.aws |
| Elevenlabs | `ELEVENLABS_API_KEY`                             |
| OpenAI     | `OPENAI_API_KEY`                                 |
| Deepgram   | `DEEPGRAM_AUTH_TOKEN`                            |
| Cartesia   | `CARTESIA_API_KEY`                            |
| Smallest   | `SMALLEST_API_KEY`                            |

</details>
&nbsp;<br>

<details>

<summary>Telephony Providers</summary><br>
These are the current supported Telephony Providers:

| Provider | Environment variable to be added in `.env` file                                                                                                                    |
|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Twilio   | `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER`|
| Plivo    | `PLIVO_AUTH_ID`, `PLIVO_AUTH_TOKEN`, `PLIVO_PHONE_NUMBER`|

</details>

## About This Project
This is a fully open-source project maintained by the community. We welcome contributions and are committed to improving the adoption of Voice AI technology.

## Extending with other Telephony Providers
To extend and add other telephony providers like Vonage, Telnyx, etc., follow these guidelines:
1. Ensure bi-directional streaming is supported by the telephony provider
2. Add a telephony-specific input handler file in `voiceai/input_handlers/telephony_providers/` with custom functions extending from the `telephony.py` class
   - This file will handle how different event packets are ingested from the telephony provider
3. Add a telephony-specific output handler file in `voiceai/output_handlers/telephony_providers/` with custom functions extending from the `telephony.py` class
   - This handles converting audio from the synthesizer to a supported format and streaming it over the websocket
4. Write a dedicated server like the example `twilio_api_server.py` provided in `local_setup/telephony_server/` to initiate calls over websockets
