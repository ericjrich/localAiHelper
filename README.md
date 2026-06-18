# e-ai — Offline Local AI for Linux

Set up a private, local AI chatbot on a Linux machine — no Docker, no cloud, no
account. `e-ai` installs and wires together [Ollama](https://ollama.com) (the
model engine) and [Open WebUI](https://github.com/open-webui/open-webui) (a
browser chat UI), pulls models for you, and can run a model right in your
terminal. It also builds a fully **portable offline archive** so you can stand
the whole stack up on a machine that has never touched the internet.

It's two single-file Bash scripts you can read end to end:

| Script | What it's for |
| --- | --- |
| `e-ai.eb` | The full toolkit: install/repair Ollama + Open WebUI, pull models, run models in the terminal, and create/restore offline archives. |
| `e-ailodal.eb` | A tiny terminal-only launcher for chatting with a model — plain, or wearing the **Lodal** personality. |

Both scripts share the same file locations, so they work together.

---

## Features

- **One-command setup** of Ollama + Open WebUI, without Docker.
- **Model management** — pull `llama3.2:3b` (text) and/or `gemma3:4b` (text **and images**), or any model by name.
- **Terminal chat** — run any installed model in a new terminal window, generic or with the **Lodal** persona injected.
- **Start/Stop toggles** for the web UI and terminal sessions, with live `[running]`/`[stopped]` status.
- **Offline archive** — bundle the engine, models, the Open WebUI installer, and your persona into one file (`.tar.zst` or `.zip`) and restore it on an air-gapped machine.
- **Self-installing archives** — a fresh machine with no copy of the script can still install from the archive via a bundled `install.sh`.
- **Lodal** — a built-in, foul-mouthed-but-helpful AI personality you can paste into Open WebUI or bake into a model.

---

## Requirements

- A Linux machine — built and tested against **Linux Mint / Ubuntu-based** distributions.
- **Internet** for the first install (to download Ollama, models, and Open WebUI). After that it runs fully offline.
- **Python 3.11 or 3.12** for Open WebUI (Mint 22 ships 3.12). Not needed if you only use the terminal chat.
- For `.tar.zst` archives: `zstd` (a `.zip` archive needs nothing extra).

No Docker required.

---

## Quick start

```bash
chmod +x e-ai.eb
./e-ai.eb
```

That opens the interactive menu. To do a full setup non-interactively:

```bash
./e-ai.eb all          # install Ollama, pull the default model, install Open WebUI
./e-ai.eb start        # start Open WebUI, then open http://localhost:8080
```

---

## The menu (`e-ai.eb`)

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
q) Quit
```

- **1) Do everything** — a checklist: tick which tasks to run (install Ollama, pull `llama3.2:3b`, pull `gemma3:4b`, install Open WebUI, create launcher), then go.
- **2) Install/repair** — submenu for Ollama or Open WebUI individually.
- **3) Pull a model** — pick `llama3.2:3b`, `gemma3:4b`, or type any model name.
- **4) Start/Stop local web UI** — starts Open WebUI in a **new terminal window** (so the menu stays usable); stops it in place when running.
- **5) Status** — what's installed and running.
- **6) Create launcher** — writes a standalone start script.
- **7) Offline AI Menu** — create or restore an offline archive (see below).
- **8) / 9) Start/Stop a model in terminal** — pick from your installed models and run it in a new window. Option 9 builds the **Lodal** persona onto the base model you choose.

### Command line

```
./e-ai.eb              Interactive menu
./e-ai.eb all          Full setup
./e-ai.eb ollama       Install/repair Ollama
./e-ai.eb model [name] Pull a model (default: llama3.2:3b)
./e-ai.eb webui        Install/repair Open WebUI
./e-ai.eb start        Start the browser web UI
./e-ai.eb status       Show status
./e-ai.eb runner       Create launcher only
./e-ai.eb archive      Create offline install archive (.tar.zst/.zip)
./e-ai.eb restore      Restore/install from an e-ai archive
```

---

## Terminal Lodal (`e-ailodal.eb`)

A simple, GUI-free way to chat in the terminal.

```bash
chmod +x e-ailodal.eb
./e-ailodal.eb
```

```
=== ollama (no gui) lodal ===
• current model: llama3.2:3b

1. Run the ollama and inject the lodal details first
2. Run the ollama without the lodal stuff
3. Install ollama (check if you already have it first)
4. Choose model (text vs vision)
     q. quit
```

Option 4 toggles which model is active (`llama3.2:3b`, `gemma3:4b`, or any name),
and that choice drives both the Lodal build and the plain run.

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
to change Lodal's behavior; both scripts pick up the change. You can use Lodal
two ways:

- **Terminal** — option 9 in `e-ai.eb` (or `e-ailodal.eb`) bakes the persona into
  a derived `lodal` model and runs it. You can then also just `ollama run lodal`.
- **Open WebUI** — paste the contents of `persona-lodal.txt` into
  *Workspace → Models → System Prompt*.

---

## Offline archive

The Offline AI Menu builds a single portable bundle that reproduces the whole
stack on a machine with no internet.

**What it contains:**

- The Ollama engine binary (and support libraries).
- The models you select (a checklist — only the chosen models' data is copied, with shared layers de-duplicated).
- An offline Python **wheelhouse** to install Open WebUI without internet.
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

The restore lays down Ollama, your models, and Open WebUI (from the wheelhouse),
drops the Lodal persona into place, and installs `e-ai.eb` into your path.

> **Requirements on the target:** same CPU architecture (x86_64) and a base
> Python matching the one recorded in the archive (Mint 22 = 3.12). A `.tar.zst`
> archive also needs `zstd` to extract.

> **Not migrated:** Open WebUI accounts, chats, and saved settings are *not*
> guaranteed to transfer — treat the restored Open WebUI as a fresh install. The
> Lodal persona travels as a text file so you can recreate it.

---

## File locations

| Path | What |
| --- | --- |
| `~/.local/share/e-local-ai/` | Data dir (`BASE_DIR`) — venv, logs, persona, stamps. |
| `~/.local/share/e-local-ai/persona-lodal.txt` | The Lodal system prompt (shared by both scripts). |
| `~/.local/share/e-local-ai/open-webui-venv/` | Open WebUI Python virtual environment. |
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

Because everything installs through these scripts, you can bake the whole stack
into a custom live ISO with [Cubic](https://launchpad.net/cubic) and boot a
machine straight into offline AI. Inside Cubic's chroot, install Ollama, pull a
model into the system model dir, and drop in the scripts; then set an autologin
into `e-ailodal.eb` (terminal flavor) or a kiosk browser to `localhost:8080`
(web UI flavor).

A few chroot-specific notes: systemd isn't running during the build (service
files are placed but activate at boot, not in the chroot), install to
system-wide paths via `E_LOCAL_AI_DIR=/opt/e-local-ai` rather than `$HOME`, and
pull models into `/usr/share/ollama/.ollama/models` (chown to `ollama:ollama`)
so the boot-time Ollama service can see them.

---

## Notes & caveats

- **CPU inference** is the default and works everywhere; GPU acceleration needs
  the relevant drivers installed and isn't required.
- Options 4, 8, and 9 open **new terminal windows** and auto-detect your
  emulator (gnome-terminal, konsole, xfce4-terminal, mate-terminal, kitty,
  alacritty, xterm). If yours isn't matched, the script tells you the command to
  run yourself.
- Plain chat needs no internet once installed. Open WebUI's document/RAG features
  may try to fetch a small embedding model on first use — do that while online if
  you need it offline later.

---

## License

_Add your license of choice here (e.g. MIT)._
