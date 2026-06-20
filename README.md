# e-ai — Offline Local AI for Linux

Set up a private, local AI chatbot on a Linux machine — no Docker, no cloud, no
account. `e-ai` installs and wires together [Ollama](https://ollama.com) (the
model engine) and [Open WebUI](https://github.com/open-webui/open-webui) (a
browser chat UI), pulls models for you, runs a model right in your terminal, and
adds fully **offline voice chat** (speak to it, hear it reply). It also builds a
**portable offline archive** so you can stand the whole stack up on a machine
that has never touched the internet.

It's a single, readable Bash script: **`e-ai.eb`**. (An earlier terminal-only
helper, `e-ailodal.eb`, has been retired — everything it did now lives in
`e-ai.eb`.)

---

## Features

- **One-command setup** of Ollama + Open WebUI, without Docker.
- **Model management** — pull `llama3.2:3b` (text) and/or `gemma3:4b` (text **and images**), or any model by name.
- **Terminal chat** — run any installed model in a new terminal window, generic or with the **Lodal** persona injected.
- **Offline voice chat** — talk to your local model and hear it talk back, fully offline (faster-whisper for speech-to-text, Piper for speech). Opens in its own terminal window, and asks you which model, personality, and voice to use each time.
- **Start/Stop toggles** for the web UI and terminal sessions, with live `[running]`/`[stopped]` status.
- **Offline archive** — bundle the engine, models, the Open WebUI installer, the voice setup, and required system packages into one file (`.tar.zst` or `.zip`) and restore it on an air-gapped machine.
- **Self-installing archives** — a fresh machine with no copy of the script can still install from the archive via a bundled `install.sh`.
- **Uninstall / start over** — tear it all back down (with optional removal of models and Ollama itself) when you want a clean slate.
- **Auto-installs missing tools** with your package manager (non-interactively, `-y`), so setup doesn't stall on a missing dependency.
- **Lodal** — a built-in, foul-mouthed-but-helpful AI personality you can paste into Open WebUI or bake into a model.

---

## Requirements

- A Linux machine — built and tested against **Linux Mint / Ubuntu-based** distributions.
- **Internet** for the first install (to download Ollama, models, Open WebUI, and the voice assets). After that it runs fully offline.
- **Python 3.11 or 3.12** for Open WebUI and voice (Mint 22 ships 3.12). Not needed if you only use the plain terminal chat.
- For `.tar.zst` archives: `zstd` (a `.zip` archive needs nothing extra).

Missing tools are installed automatically (non-interactively, with `-y`) — you'll
just be asked for your `sudo` password when system packages are involved.

No Docker required.

---

## Quick start

```bash
chmod +x e-ai.eb
./e-ai.eb
```

That opens the interactive menu. To do a full setup non-interactively:

```bash
./e-ai.eb all          # Ollama + default model + Open WebUI + voice + launcher
./e-ai.eb start        # start Open WebUI, then open http://localhost:8080
```

---

## The menu

```
e-ai: local offline chatbot setup

1) Do everything
2) Install/repair
3) Pull a model
4) Start/Stop local web UI                         [stopped]
5) Status
6) Create launcher
7) Offline AI Menu
8) Start/Stop A Model In Terminal (Generic)        [stopped]
9) Start/Stop A Model In Terminal (Inject Lodal)   [stopped]
v) Voice chat (offline)
u) Uninstall / start over
q) Quit
```

- **1) Do everything** — a checklist: tick which tasks to run (install Ollama, pull `llama3.2:3b`, pull `gemma3:4b`, install Open WebUI, set up voice support, create launcher), then go.
- **2) Install/repair** — submenu for Ollama or Open WebUI individually.
- **3) Pull a model** — pick `llama3.2:3b`, `gemma3:4b`, or type any model name.
- **4) Start/Stop local web UI** — starts Open WebUI in a **new terminal window** (so the menu stays usable); stops it in place when running.
- **5) Status** — what's installed and running.
- **6) Create launcher** — writes a standalone start script.
- **7) Offline AI Menu** — create or restore an offline archive (see below).
- **8) / 9) Start/Stop a model in terminal** — pick from your installed models and run it in a new window. Option 9 builds the **Lodal** persona onto the base model you choose.
- **v) Voice chat** — offline speech in/out (see below).
- **u) Uninstall** — remove e-ai, optionally Ollama and models, to start fresh.

### Command line

```
./e-ai.eb              Interactive menu
./e-ai.eb all          Full setup (Ollama + model + Open WebUI + voice + launcher)
./e-ai.eb ollama       Install/repair Ollama
./e-ai.eb model [name] Pull a model (default: llama3.2:3b)
./e-ai.eb webui        Install/repair Open WebUI
./e-ai.eb start        Start the browser web UI
./e-ai.eb status       Show status
./e-ai.eb runner       Create launcher only
./e-ai.eb voice        Offline voice chat (mic -> model -> speakers)
./e-ai.eb archive      Create offline install archive (.tar.zst/.zip)
./e-ai.eb restore      Restore/install from an e-ai archive
./e-ai.eb uninstall    Remove e-ai (optionally Ollama + models) to start over
```

---

## Models

| Model | Use | Notes |
| --- | --- | --- |
| `llama3.2:3b` | Text chat | Small and fast; the default. |
| `gemma3:4b` | Text **+ images** | Multimodal — paste an image path into your prompt to have it look at screenshots, photos, diagrams, etc. Needs Ollama 0.6+. |

You can pull any other Ollama model by name from the "Pull a model" menu or
`./e-ai.eb model <name>`.

---

## Lodal

**Lodal** is a built-in AI personality: a foul-mouthed, sarcastic anti-hero with
a big mouth and a bigger heart, convinced he's a real person. He's the "survivor
voice" — sharp, funny, a little lewd, and built around a self-respect code
(protect your peace, move with strategy not panic, save your fire for what
matters). Crude on the surface, genuinely helpful underneath, and weirdly
well-educated — he drops real knowledge between the jokes, especially on Linux,
Bash, coding, and troubleshooting.

The persona text lives at `~/.local/share/e-local-ai/persona-lodal.txt`. Edit it
to change Lodal's behavior. You can use Lodal three ways:

- **Terminal** — option 9 bakes the persona into a derived `lodal` model and runs
  it in a new window. You can then also just `ollama run lodal`.
- **Voice** — choose "Lodal" when voice chat asks for a personality.
- **Open WebUI** — paste the contents of `persona-lodal.txt` into
  *Workspace → Models → System Prompt*.

---

## Voice chat (offline)

Talk to your local model and have it talk back — entirely offline. The pipeline
is **mic → faster-whisper (speech-to-text) → local model → Piper (text-to-speech)
→ speakers**.

It's **always installed** (small next to the model weights) but **only loaded
when you use it** — no microphone access, no speech models in memory, and no
performance impact during normal text chat.

```bash
./e-ai.eb voice      # or pick "Voice chat (offline)" from the menu
```

When you start it, voice chat **asks you four quick questions**, then launches
the session in **its own terminal window**:

1. **Which model?** — chosen from your installed models (or type any name).
2. **Personality?** — inject **Lodal**, type a **custom prompt** (or point at a
   `.txt` file), or go **generic** (no system prompt).
3. **Voice?** — **Female** (`en_US-amy-medium`), **Male** (`en_US-ryan-medium`),
   **Neutral** (`en_US-lessac-medium`), or any other Piper voice by name.
4. **Mode?** — **push-to-talk** (Enter to start/stop) or **continuous**
   (voice-activated: speak, pause, it replies).

Settings and defaults live in `~/.local/share/e-local-ai/voice/voice.conf`
(mode, Whisper model size, Piper voice, persona, model, mic device). The picker
overrides them per session.

| Piece | Tool | What's pre-downloaded |
| --- | --- | --- |
| Speech-to-text | [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | both `base` and `small` models, so switching is instant |
| Text-to-speech | [Piper](https://github.com/rhasspy/piper) | the full **English voice set** (US + GB, all quality tiers) |
| Model | any installed Ollama model | `llama3.2:3b` by default |

Installing voice support pulls the **whole English voice set** and **both Whisper
sizes** up front, so every choice in the picker works offline with no
mid-session downloads. The offline archive captures the entire voice setup — the
venv (as an offline wheelhouse), the Whisper cache, every installed Piper voice,
and your `voice.conf` — so a restored machine has working voice chat with no
internet.

> Voice needs system audio libraries (`libportaudio2`, `alsa-utils`), which the
> installer adds automatically. Inference is CPU-based by default. Wake-word
> support is planned for a later release.

> **Open WebUI voice:** Open WebUI has its own built-in speech settings. Point
> them at the same local Whisper/Piper this installs, rather than expecting voice
> toggles to appear from this tool.

---

## Offline archive

The Offline AI Menu builds a single portable bundle that reproduces the whole
stack on a machine with no internet.

**What it contains:**

- The Ollama engine binary (and support libraries).
- The models you select (a checklist — only the chosen models' data is copied, with shared layers de-duplicated).
- An offline Python **wheelhouse** to install Open WebUI without internet.
- The voice setup, if installed — the voice venv (as an offline wheelhouse), Whisper cache, every installed Piper voice, and `voice.conf`.
- The operational **system packages** as offline `.deb` files (audio libs, Python venv/dev, build tools) so the target needs no internet to become functional.
- `persona-lodal.txt` — your ready-to-paste Lodal prompt.
- A copy of `e-ai.eb` plus a bootstrap `install.sh`.
- Version/manifest metadata.

**Formats:** `.tar.zst` (preferred — Linux-native, smaller, faster) with
automatic fallback to `.zip`.

**Restoring on the target machine:**

- **If it already has `e-ai.eb`:** run it → *Offline AI Menu → Restore*, or
  `./e-ai.eb restore`.
- **If it has nothing:** extract the archive and run the bundled bootstrap:

  ```bash
  # .zip
  unzip e-ai.zip && cd e-ai-offline && bash install.sh
  # .tar.zst
  tar -xf e-ai-offline-bundle.tar.zst && cd e-ai-offline && bash install.sh
  ```

The restore installs the bundled system packages, lays down Ollama, your models,
and Open WebUI (from the wheelhouse), rebuilds the voice environment, drops the
Lodal persona into place, and installs `e-ai.eb` into your path.

> **Requirements on the target:** same CPU architecture (x86_64) and a base
> Python matching the one recorded in the archive (Mint 22 = 3.12). A `.tar.zst`
> archive also needs `zstd` to extract. System `.deb` capture/restore is apt-only
> and assumes a similar Mint/Ubuntu release.

> **Not migrated:** Open WebUI accounts, chats, and saved settings are *not*
> guaranteed to transfer — treat the restored Open WebUI as a fresh install. The
> Lodal persona travels as a text file so you can recreate it.

---

## Uninstall / start over

Menu option **u** (or `./e-ai.eb uninstall`) tears the stack back down. It always
confirms first, then:

- Stops Open WebUI and the Ollama server.
- Removes the e-ai **data directory** (`~/.local/share/e-local-ai` — both venvs,
  all voice data and voices, persona, logs) and the **launcher**.

Then it asks two separate, optional questions:

- **Delete downloaded Ollama models?** — default **no**, so you keep your
  multi-GB models and a fresh `Do everything` is quick.
- **Uninstall Ollama entirely?** — default **no**; yes removes the Ollama binary,
  its systemd service, model stores, and the `ollama` user.

Cancelling at the first prompt leaves everything untouched.

---

## File locations

| Path | What |
| --- | --- |
| `~/.local/share/e-local-ai/` | Data dir (`BASE_DIR`) — venvs, logs, persona, voice. |
| `~/.local/share/e-local-ai/persona-lodal.txt` | The Lodal system prompt. |
| `~/.local/share/e-local-ai/open-webui-venv/` | Open WebUI Python virtual environment. |
| `~/.local/share/e-local-ai/voice-venv/` | Voice (faster-whisper + Piper) virtual environment. |
| `~/.local/share/e-local-ai/voice/` | Voice config, Piper voices, Whisper cache, helper. |
| `~/a-me/bin/` | Installed launcher and tools (`BIN_DIR`). |
| `~/a-me/bin/e-local-ai-run.eb` | The generated launcher. |
| `~/.ollama/models/` | Ollama model store. |

### Environment variables

| Variable | Default | Effect |
| --- | --- | --- |
| `WEBUI_PORT` | `8080` | Port Open WebUI listens on. |
| `E_LOCAL_AI_DIR` | `~/.local/share/e-local-ai` | Override the data directory. |

---

## Building a live-USB appliance (Cubic)

Because everything installs through this script, you can bake the whole stack
into a custom live ISO with [Cubic](https://launchpad.net/cubic) and boot a
machine straight into offline AI. Inside Cubic's chroot, install Ollama, pull a
model into the system model dir, and drop in the script; then set an autologin
into a terminal model run or a kiosk browser to `localhost:8080` (web UI flavor).

A few chroot-specific notes: systemd isn't running during the build (service
files are placed but activate at boot, not in the chroot), install to
system-wide paths via `E_LOCAL_AI_DIR=/opt/e-local-ai` rather than `$HOME`, and
pull models into `/usr/share/ollama/.ollama/models` (chown to `ollama:ollama`)
so the boot-time Ollama service can see them.

---

## Notes & caveats

- **CPU inference** is the default and works everywhere; GPU acceleration needs
  the relevant drivers installed and isn't required.
- Options **4, 8, 9, and v** open **new terminal windows** and auto-detect your
  emulator (gnome-terminal, konsole, xfce4-terminal, mate-terminal, tilix, kitty,
  alacritty, xterm). If none is found, `xterm` is installed automatically.
- The mic and speaker behavior in voice chat depends on your system audio
  (PulseAudio/PipeWire/ALSA). If playback is silent, make sure `aplay`/`paplay`
  works and your default output device is set.
- Plain chat needs no internet once installed. Open WebUI's document/RAG features
  may try to fetch a small embedding model on first use — do that while online if
  you need it offline later.

---

## License

_Add your license of choice here (e.g. MIT)._
