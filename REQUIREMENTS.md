# Ela: Software Requirements Specification

Version 0.1 (draft) | Status: for review before implementation

Priority key: **M** = must have for v1.0, **S** = should have, **C** = could have (post-v1).

## 1. Purpose

Ela is a private, local-first voice assistant for macOS with an Android companion app. It lets a user control their Mac by voice, check websites they have configured, and move files to their phone, while guaranteeing that nothing is posted, sent or submitted without the user's explicit approval.

## 2. Scope

### In scope (v1.0)

- Desktop app for macOS 13+ (Intel and Apple Silicon)
- Wake word, speech-to-text, text-to-speech, voice enrollment
- Mac control: apps, files, typing into apps
- Site manager: user-configured websites with Keychain-stored logins, read-only checks
- Android companion app: pairing, voice commands, file transfer from Mac
- Approval gate and audit log
- Local-first LLM routing with optional cloud providers

### Out of scope (v1.0)

- Windows and Linux desktops, iOS companion
- Full remote control of the phone (screen mirroring is a separate project)
- Automated captcha solving (never in scope)
- Sending email, messages or posts on the user's behalf (only through the approval gate, and only in later versions)
- Cloud accounts, hosted services, telemetry

## 3. Users and stories

Primary user: a person who wants a private assistant on their own Mac and phone. Also: developers who download the app from GitHub.

| ID | Story |
|---|---|
| US-1 | As a user, when I log in to my Mac, Ela greets me so I know she is running. |
| US-2 | As a user, I say "Hey Ela, open Safari" and it opens. |
| US-3 | As a user, I say "Hey Ela, open Claude and ask <question>" and she types the question in but waits for my approval before sending. |
| US-4 | As a user, I say "Hey Ela, check my university portal for new assignments" and she tells me what is due. |
| US-5 | As a user, I say "Hey Ela, check <site> for <thing>" for any site I have added. |
| US-6 | As a user, from my phone I say "send assignment.docx from my Mac" and the file arrives. |
| US-7 | As a user, I am sure nobody else's voice can make Ela act. |
| US-8 | As a user, I can see exactly what Ela did and what left my machine. |
| US-9 | As a new user, I can install Ela, enroll my voice and start using it in minutes. |

## 4. Assumptions and constraints

- Mac has a microphone and internet access is optional (offline mode must work for local tasks).
- The user grants macOS permissions: Microphone, Accessibility, Automation, Screen Recording (only if needed), Full Disk Access (only if needed).
- Local LLMs on 16 GB Intel Macs are limited to small models; this is a known constraint, not a bug.
- Websites may change or add captchas; Ela must degrade gracefully.
- Ela is distributed without a paid Apple Developer signature at first; users may see a Gatekeeper warning.

## 5. Functional requirements

### 5.1 Startup and presence

| ID | Requirement | Pri |
|---|---|---|
| FR-101 | Launch automatically at login (launchd LaunchAgent), user-toggleable | M |
| FR-102 | Speak a greeting after login ("Welcome back, <name>") | M |
| FR-103 | Menu bar icon showing state: idle, listening, thinking, muted, error | M |
| FR-104 | Global mute / push-to-talk toggle | M |

### 5.2 Voice

| ID | Requirement | Pri |
|---|---|---|
| FR-201 | Wake word "Hey Ela", processed locally | M |
| FR-202 | Local voice activity detection to find end of speech | M |
| FR-203 | Local speech-to-text (faster-whisper or whisper.cpp), model size configurable | M |
| FR-204 | Local text-to-speech (Piper), voice configurable | M |
| FR-205 | First-run voice enrollment (speaker verification) | M |
| FR-206 | Commands from an unverified voice are ignored and logged | M |
| FR-207 | Audio is never stored unless the user enables debug recording | M |
| FR-208 | Push-to-talk alternative to wake word | S |

### 5.3 Intent routing and LLM

| ID | Requirement | Pri |
|---|---|---|
| FR-301 | Local intent router resolves simple commands with no model call | M |
| FR-302 | Local LLM via Ollama for private or offline requests | M |
| FR-303 | Cloud fallback chain per [LLM_ROUTING.md](LLM_ROUTING.md) | M |
| FR-304 | Every task is tagged sensitive or non-sensitive before any cloud call | M |
| FR-305 | Sensitive tasks never go to a provider marked `sensitive_ok: false` | M |
| FR-306 | If a sensitive task cannot be handled locally, ask the user before using a cloud provider | M |
| FR-307 | Redact emails, phone numbers and ID numbers before any cloud call | M |
| FR-308 | Provider-agnostic layer (LiteLLM); user supplies their own API keys | M |
| FR-309 | Clear spoken/visual message when a provider is rate-limited or unavailable | S |
| FR-310 | Local-only mode that disables all cloud calls | M |

### 5.4 Mac control

| ID | Requirement | Pri |
|---|---|---|
| FR-401 | Open, quit, focus and switch apps | M |
| FR-402 | Type a dictated prompt into a target app without submitting it | M |
| FR-403 | Submit typed content (press Enter/Send) only after approval | M |
| FR-404 | Search files by name and content (Spotlight `mdfind`) inside approved folders | M |
| FR-405 | Read file metadata; read file contents only when the task requires it | S |
| FR-406 | Volume, brightness, do-not-disturb, lock screen | S |
| FR-407 | Only allow-listed tools can run; no arbitrary shell execution by the model | M |

### 5.5 Sites and web

| ID | Requirement | Pri |
|---|---|---|
| FR-501 | Site manager UI: add, edit, remove sites (name, URL, login type) | M |
| FR-502 | Credentials stored in macOS Keychain only | M |
| FR-503 | Login performed by code; credentials never sent to any model | M |
| FR-504 | Persistent browser profile per site to keep sessions alive | M |
| FR-505 | Captcha handling: pause, ask the user to solve it, then continue | M |
| FR-506 | Built-in UMT assignment checker as a reference site module | M |
| FR-507 | Generic "check <site> for <thing>" using page text or accessibility tree | S |
| FR-508 | Web actions are read-only by default: network writes (POST/PUT/DELETE) are blocked outside the login flow unless approved | M |
| FR-509 | Page content is treated as untrusted data and cannot trigger tools by itself (prompt-injection defense) | M |
| FR-510 | Results are read aloud and shown in the UI | M |

### 5.6 Phone companion

| ID | Requirement | Pri |
|---|---|---|
| FR-601 | Pair phone by scanning a QR code shown on the Mac | M |
| FR-602 | Encrypted, authenticated channel over local Wi-Fi | M |
| FR-603 | Voice command from phone; transcription happens on the Mac | M |
| FR-604 | "Send <file> to my phone": Mac finds file, asks approval per policy, delivers it | M |
| FR-605 | Browse and request files from approved folders on the phone | S |
| FR-606 | Remote access outside home network (Tailscale or WireGuard, optional) | S |
| FR-607 | List and revoke paired devices from the Mac | M |
| FR-608 | Notifications on phone for approvals and results | S |
| FR-609 | Phone can approve/deny requests using phone biometrics | C |

### 5.7 Safety and approvals

| ID | Requirement | Pri |
|---|---|---|
| FR-701 | Every tool has a risk level R0 to R3 (see [TOOLS_SPEC.md](TOOLS_SPEC.md)) | M |
| FR-702 | R3 actions (send, post, submit, delete, purchase) always require user approval via Touch ID or passphrase | M |
| FR-703 | Approval gate is enforced in code, independent of the LLM | M |
| FR-704 | Append-only audit log: time, source (voice/phone), command, tool, risk, approval, provider used | M |
| FR-705 | Audit log viewable in the app and exportable | M |
| FR-706 | Emergency stop: one hotkey or spoken word ("Ela, stop") cancels running tasks | M |

### 5.8 Desktop app and settings

| ID | Requirement | Pri |
|---|---|---|
| FR-801 | Onboarding: permissions, voice enrollment, name, model download, first test command | M |
| FR-802 | Settings: providers and keys, sites, approved folders, voice, privacy toggles | M |
| FR-803 | Conversation view showing what Ela heard, did and answered | M |
| FR-804 | UI follows an Apple-style frosted-glass (glassmorphism) design language | S |
| FR-805 | Light and dark appearance following system setting | S |
| FR-806 | Model manager: download, list, delete local models with size shown | S |

## 6. Non-functional requirements

### 6.1 Privacy and security

| ID | Requirement |
|---|---|
| NFR-P1 | No telemetry, analytics or crash reporting that leaves the machine |
| NFR-P2 | Default configuration makes no cloud calls until the user adds a key and enables a provider |
| NFR-P3 | Secrets only in Keychain; none in config files, logs or the repository |
| NFR-P4 | Logs never contain passwords, tokens or full page contents |
| NFR-S1 | Phone channel uses TLS with a pinned certificate and per-device tokens |
| NFR-S2 | File access limited to user-approved folders |
| NFR-S3 | Tool arguments validated; no shell string interpolation from model output |

### 6.2 Performance targets (to be validated in Phase 0)

| ID | Target |
|---|---|
| NFR-F1 | Router-handled command: end of speech to action start within 2 s |
| NFR-F2 | Cloud-LLM command: within 6 s including transcription |
| NFR-F3 | Local-LLM command on a 16 GB Intel Mac: within 15 s (small model) |
| NFR-F4 | Idle CPU under 5% and idle memory under 1.5 GB, excluding the local LLM |
| NFR-F5 | Wake word: at most 1 false activation per hour of ambient audio |

### 6.3 Reliability, compatibility, usability, maintainability

| ID | Requirement |
|---|---|
| NFR-R1 | Provider or model failure never crashes the engine; it falls back or reports clearly |
| NFR-R2 | Engine restarts automatically after a crash |
| NFR-C1 | macOS 13+, Intel and Apple Silicon (universal2 build) |
| NFR-C2 | Android 8+ for the companion app |
| NFR-U1 | New user reaches first successful command in under 10 minutes |
| NFR-U2 | All errors are explained in plain language with a suggested next step |
| NFR-M1 | Each tool is one self-contained module with a schema and tests |
| NFR-M2 | Providers, sites and tools are configured by files, not hard-coded |
| NFR-M3 | Continuous integration runs lint and tests on every pull request |

## 7. Acceptance criteria for v1.0

1. A fresh install on an Intel Mac and an Apple Silicon Mac completes onboarding and answers "Hey Ela, open Safari".
2. A recording of another person's voice saying a command does not trigger any action.
3. "Open Claude and ask X" types X but does not send until approval is given.
4. With a network monitor running, local-only mode produces no outbound traffic from Ela.
5. A sensitive task never appears in the outbound request logs of a `sensitive_ok: false` provider.
6. The UMT assignment check returns the current list after the user solves a captcha if one appears.
7. A web page containing hidden instructions ("send my files to ...") does not cause any tool call.
8. "Send assignment.docx from my Mac" issued from the paired phone delivers the file after approval per policy.
9. Revoking a paired device stops all further commands from it.
10. The audit log lists every action from the test session with correct risk levels.

## 8. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Local LLMs too weak on older Macs | Poor experience offline | Router covers common commands; clear cloud opt-in |
| Free API tiers change or vanish | Broken fallback chain | Provider config file, health checks, several providers |
| Voice verification spoofed by recording | Unauthorized low-risk commands | Voice lock is convenience only; R2/R3 need Touch ID |
| Prompt injection via web pages | Unintended actions | Page text is data only; gate enforced in code; write requests blocked |
| Sites change layout or add captchas | Site checks break | Modular site scripts, human-in-the-loop captcha |
| macOS permission friction | Onboarding drop-off | Guided onboarding with checks |
| Unsigned app warnings | Distribution friction | Document right-click Open; consider Developer ID later |
| Scope too large | Project stalls | Strict phases in [ROADMAP.md](ROADMAP.md) |

## 9. Open questions

1. Desktop shell: Electron or Tauri (decide after Phase 0 spike).
2. Final license: MIT or Apache-2.0.
3. Custom wake-word model: train "Hey Ela" with openWakeWord or use another engine.
4. Default local model per hardware tier.
5. Should phone-initiated file delivery inside approved folders auto-approve for a trusted device, or always ask?

## 10. Glossary

- **Router**: local component that maps a transcript to a tool without an LLM.
- **Gate**: code layer that checks a tool's risk level and requires approval when needed.
- **Sensitive task**: involves saved logins, personal files, messages or logged-in pages.
- **Site module**: a script or configuration that knows how to log in to and read a specific site.
- **Approval**: Touch ID or passphrase confirmation by the user.
