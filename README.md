# voice-agent

A free, low-latency voice agent that talks like a human: it streams replies,
stops the moment you interrupt, and responds to how you feel.

## What we're building

You speak in the browser → it hears you → thinks → speaks back in about 1 second.

```
Mic ──▶ VAD ──▶ Speech-to-Text ──▶ LLM ──▶ Text-to-Speech ──▶ Speaker
        (Silero)  (NVIDIA Parakeet)  (NVIDIA NIM)  (NVIDIA Magpie / Kokoro)
          ▲                                                   │
          └──────── you start talking = agent stops ◀─────────┘
```

| Goal | How |
|---|---|
| **Low latency** | Everything streams. TTS starts speaking on the first sentence, not the full reply. Small LLM, 1–2 sentence answers. |
| **Interruptions** | Pipecat cancels TTS as soon as the VAD hears you speaking. |
| **Human, emotional tone** | System prompt: casual, short, reacts to the user's feelings. |
| **Reliable** | Retry once on API errors, then speak a short filler ("one sec…") instead of going silent. |
| **Weak server is fine** | Only the orchestrator runs locally. Speech-to-text, the LLM and TTS all run on NVIDIA's cloud. |

## Free tools

| Part | Tool | Cost |
|---|---|---|
| Pipeline / orchestration | [Pipecat](https://github.com/pipecat-ai/pipecat) | Free, open source |
| Voice activity detection | Silero VAD (bundled with Pipecat) | Free, runs on CPU |
| Speech-to-text | NVIDIA Parakeet via [build.nvidia.com](https://build.nvidia.com) | Free API key |
| LLM | NVIDIA NIM, e.g. `meta/llama-3.1-8b-instruct` | Free API key |
| Text-to-speech | NVIDIA Magpie TTS (same key), or Kokoro (local, CPU) | Free |
| Browser audio | Pipecat SmallWebRTC transport | Free, no third-party account |

## Requirements

- Python **3.11–3.13** (avoid 3.14 for now because some audio and ML dependencies may not have builds for it yet)
- A free NVIDIA API key from https://build.nvidia.com
- A browser with a microphone

## Setup

```bash
uv venv --python 3.12        # uv downloads Python 3.12 if you don't have it
source .venv/bin/activate
uv pip install -r requirements.txt
cp .env.example .env        # then paste your NVIDIA_API_KEY
```

## Latency tips

- Keep the model small (8B-class or smaller). Bigger models add noticeable delay.
- Cap replies at 1–2 sentences in the system prompt.
- Run the server close to NVIDIA's region (US) if you can.
- Rate limit: one conversation uses only a few requests per minute, so a 430 RPM key is plenty.

## Status

- [x] Project setup, requirements, README
- [ ] `bot.py`, the Pipecat pipeline (STT → LLM → TTS with interruptions)
- [ ] System prompt for human, emotional tone
- [ ] Retry and filler-line fallback
