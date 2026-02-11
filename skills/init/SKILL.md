---
name: init
description: Scaffold a new Pipecat project with guided setup
---

Scaffold a new Pipecat project by collecting configuration from the user and running `pc init` in non-interactive mode.

## Arguments

```
/init [--output <PATH>]
```

- `--output` (optional): Directory where the project will be created. Defaults to the current directory.

## Prerequisites

Check if `pc` is installed by running `pc --version`. If not installed, tell the user to install it with `uv tool install pipecat-ai-cli` and stop.

## Discover Available Options

Before asking the user any questions, run `pc init --list-options` to get the current valid values for all fields. The output is JSON:

```json
{
  "bot_type": ["web", "telephony"],
  "transports": {
    "web": ["daily", "smallwebrtc"],
    "telephony": ["twilio", "telnyx", ...]
  },
  "stt": ["deepgram_stt", "openai_stt", ...],
  "llm": ["openai_llm", "anthropic_llm", ...],
  "tts": ["cartesia_tts", "elevenlabs_tts", ...],
  "realtime": ["openai_realtime", "gemini_live_realtime", ...],
  "video": ["heygen_video", "tavus_video", "simli_video"]
}
```

Use this data to populate the choices in every question below. Do NOT hardcode service lists — always use the values from `--list-options`.

## Configuration Flow

Walk through the following questions to build the project configuration. Use AskUserQuestion for each step. After collecting all answers, show a summary and run the command.

When using AskUserQuestion, present 3-4 popular options and let the user pick "Other" to specify any valid service from the full list. Suggested popular options:
- STT: deepgram_stt, openai_stt, groq_stt
- LLM: openai_llm, anthropic_llm, google_gemini_llm
- TTS: cartesia_tts, elevenlabs_tts, openai_tts
- Realtime: openai_realtime, gemini_live_realtime, ultravox

### Step 1: Project Name

Ask the user for a project name. This will be used as the directory name and project identifier.

### Step 2: Bot Type

Ask the user to choose a bot type:
- **Web/Mobile** (`web`) - Browser or mobile app
- **Telephony** (`telephony`) - Phone calls

### Step 3: Client Framework (web only)

If the bot type is `web`, ask the user to choose a client framework:
- **React** (`react`)
- **Vanilla JS** (`vanilla`)
- **None** (`none`) - Server only, no client generated

If the user chose React, ask which dev server:
- **Vite** (`vite`)
- **Next.js** (`nextjs`)

Skip this step entirely for telephony bots.

### Step 4: Transport

Ask the user to choose a transport provider. Use the transports from `--list-options` filtered by the selected bot type.

If the user chose a `daily_pstn` transport, ask for mode:
- Dial-in (receive calls) → use `--daily-pstn-mode dial-in`
- Dial-out (make calls) → use `--daily-pstn-mode dial-out`

If the user chose a `twilio_daily_sip` transport, ask for mode:
- Dial-in (receive calls) → use `--twilio-daily-sip-mode dial-in`
- Dial-out (make calls) → use `--twilio-daily-sip-mode dial-out`

Then ask if they want to add an additional transport for local testing. This is common — e.g. a telephony bot that also supports WebRTC for development.

### Step 5: Pipeline Mode

Ask the user to choose a pipeline architecture:
- **Cascade** (`cascade`) - STT → LLM → TTS pipeline
- **Realtime** (`realtime`) - Speech-to-speech model

### Step 6: AI Services

**If cascade mode**, ask the user to choose each service using the options from `--list-options`:
1. Speech-to-Text (STT)
2. Language Model (LLM)
3. Text-to-Speech (TTS)

**If realtime mode**, ask the user to choose a realtime service from the options.

### Step 7: Features

Show the user the default feature settings and ask if they want to customize:

**Defaults:**
- Audio recording: No
- Transcription logging: No
- Video avatar service: None
- Video input: No (web only)
- Video output: No (web only)
- Observability: No

If they want to customize, ask about each feature. For video avatar service (web bots only), use the video options from `--list-options`.

If a video avatar service is selected, video output is automatically enabled.

### Step 8: Deployment

Ask if they want to generate Pipecat Cloud deployment files (Dockerfile, pcc-deploy.toml). Default is yes.

If deploying to cloud, ask if they want to enable Krisp noise cancellation. Default is no.

## Building the Command

After collecting all answers, build the `pc init` command using non-interactive flags:

```
pc init \
  --name <project_name> \
  --bot-type <web|telephony> \
  --transport <transport> \
  --mode <cascade|realtime> \
  [--stt <service>] \
  [--llm <service>] \
  [--tts <service>] \
  [--realtime <service>] \
  [--video <service>] \
  [--client-framework <react|vanilla|none>] \
  [--client-server <vite|nextjs>] \
  [--daily-pstn-mode <dial-in|dial-out>] \
  [--twilio-daily-sip-mode <dial-in|dial-out>] \
  [--recording | --no-recording] \
  [--transcription | --no-transcription] \
  [--video-input | --no-video-input] \
  [--video-output | --no-video-output] \
  [--deploy-to-cloud | --no-deploy-to-cloud] \
  [--enable-krisp | --no-enable-krisp] \
  [--observability | --no-observability] \
  --output <output_dir>
```

For multiple transports, repeat the `--transport` flag (e.g. `--transport twilio --transport smallwebrtc`).

## Confirmation

Before running the command, show the user a summary of their choices:
- Project name
- Bot type
- Client framework (if web)
- Transport(s)
- Pipeline mode and services
- Features enabled
- Deployment target

Ask the user to confirm before proceeding. If they want to change something, go back and re-ask that specific question.

## Running the Command

Run the `pc init` command. Use `--output` to specify the output directory (from the `--output` argument, or default to `./<project_name>`).

If the command succeeds, show the user what was generated and suggest next steps:
1. `cd <project_name>/server`
2. Copy `.env.example` to `.env` and fill in API keys
3. Run the bot

If deploying to cloud, also mention they can use `/pipecat-cloud:deploy` to deploy.

## Error Handling

- If `pc init` fails with validation errors, show the error and help the user fix their choices.
- If the output directory already exists and is not empty, warn the user before proceeding.
