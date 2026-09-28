# Ela -- My personal assistant is in making!

**A private, local-first voice assistant for macOS, with an Android companion app.**

Say "Hey Ela", and she opens apps, finds files, checks websites you have set up (your university portal, for example) and sends files to your phone. She never posts, sends or submits anything unless you approve it.

> Status: **pre-alpha. Documentation phase, no code yet.** See [docs/ROADMAP.md](docs/ROADMAP.md).

## Principles

1. **Private by default.** Speech recognition, wake word, voice lock and text-to-speech run on your Mac. A local model handles what it can. Cloud models are optional, and personal data never goes to a provider you have not approved.
2. **You approve anything outward.** Reading, opening and searching are automatic. Sending, posting, submitting and deleting always need your approval (Touch ID or passphrase). This is enforced in code, not by prompting the model.
3. **Credentials never reach the model.** Logins are stored in the macOS Keychain and typed into pages by Ela's code. The model only sees "logged in".
4. **Bring your own keys.** No accounts, no servers, no telemetry.
5. **A real app.** Download, install, enroll your voice, talk.

## Planned features

| Area | What Ela does |
|---|---|
| Presence | Greets you at login ("Welcome back"), lives in the menu bar |
| Voice | Wake word "Ela", local speech-to-text, local text-to-speech |
| Mac control | Open/quit/focus apps, type a prompt into an app, search files |
| Websites | Open any website you configure and check what you ask (e.g. new assignments) |
| Phone | Android app paired by QR code: talk to Ela from your phone, pull files from your Mac |
| Safety | Voice lock, approval gate, audit log, sensitive/non-sensitive routing |
| LLM | Local router, then Ollama, then free cloud providers, then Gemini (non-sensitive only) |

## Documentation

| Document | Purpose |
|---|---|
| [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) | Full requirements (scope, functional, non-functional, acceptance criteria) |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Components, data flow, technology stack |
| [docs/SECURITY_AND_PRIVACY.md](docs/SECURITY_AND_PRIVACY.md) | Threat model, approval gate, credential handling |
| [docs/LLM_ROUTING.md](docs/LLM_ROUTING.md) | Model fallback chain and sensitivity rules |
| [docs/TOOLS_SPEC.md](docs/TOOLS_SPEC.md) | Tool contract, risk levels, v1 tool list |
| [docs/PHONE_PROTOCOL.md](docs/PHONE_PROTOCOL.md) | Pairing, messages, file transfer |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Phases, milestones, definition of done |
| [docs/DECISIONS.md](docs/DECISIONS.md) | Architecture decision records |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute |

## Planned repository layout

```
ela/
  app/          Desktop app UI (Electron or Tauri, decided in Phase 0)
  engine/       Python engine: voice, router, LLM layer, tools, gate
    voice/      wake word, VAD, STT, TTS, speaker verification
    brain/      intent router, LLM providers, tool-calling loop
    tools/      one module per tool
    gate/       risk levels, approvals, audit log
    sites/      site manager, Playwright automation
    phone/      pairing and channel server
  android/      Flutter companion app
  docs/
  LICENSE
```

## Requirements to run (target)

- macOS 13 (Ventura) or later, Intel or Apple Silicon
- 16 GB RAM recommended for local models (8 GB works with smaller models)
- Microphone
- Android 8+ for the companion app (optional)

## License

To be decided before first release (MIT or Apache-2.0 proposed).
