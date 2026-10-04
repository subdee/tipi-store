# SUB/WAVE

**A personal internet radio station.** One Icecast stream, one broadcast. Every listener hears the same thing at the same time. An AI DJ picks the tracks from your own music library and talks between them — station idents, time checks, the weather, a quick intro for whatever is going out next. Ask for music in plain language and the DJ works out what you meant.

It's *radio*, not a playlist. No per-listener shuffle, no skip button, no "up next for you". You tune in and hear whatever is on.

## Core features

- **One shared Icecast stream** — a single mount everyone connects to, in MP3 (always on) plus optional Opus, AAC and lossless FLAC, each toggleable from the admin console
- **An AI DJ that picks and talks** — curates the rotation, writes intros, reads station IDs, the time, the weather and the news
- **Plain-language requests** — "play something more upbeat" or "anything by Radiohead" works, from the web player or over the API
- **Your own library** — tracks come from Navidrome or any Subsonic-compatible server; nothing is streamed from an external catalogue
- **Swappable LLM provider** — Ollama, Anthropic, OpenAI, Google, DeepSeek, OpenRouter or any OpenAI-compatible server, changed from the admin UI with no redeploy, with an optional daily token budget
- **Local voices out of the box** — Piper and the multilingual Kokoro run inside the controller, so the DJ can talk with nothing else installed; OpenAI and ElevenLabs are available as cloud engines, or point it at your own TTS endpoint
- **Multiple DJ personas** — up to 24 in the roster, each with its own voice and writing style, and up to three guest co-hosts per show
- **Scheduled shows** — a 24×7 grid where each slot has its own persona, mood and skills, or anchors to a Navidrome playlist
- **Pluggable skills** — the between-track segments (weather, news, traffic, your own) are editable files under the state dir, rewritable from the admin console with no redeploy
- **Ending-aware transitions** — the bundled analyzer measures BPM, key, loudness and how each track actually ends, so crossfades size themselves to the material and the DJ doesn't talk over a sung intro
- **Playlist builder** — describe a playlist in plain language and it resolves into a recipe that keeps topping itself up as new music lands
- **Library Observatory** — a full-screen map of every tagged track at `/observatory`, placed by genre and lit by energy
- **Tune in anywhere** — the web player and PWA, the native iOS/Android/desktop apps, or paste `/listen.pls` into VLC, Sonos, moOde or a car receiver
- **Private station mode** — hide the player behind a prompt and/or require listener auth on every stream mount
- **Scrobbling** — every spin can report to Last.fm and ListenBrainz, including self-hosted instances

## First run

Open the app and go to **`/onboarding`**. Sign in with the admin username and password you set here, then the wizard collects your Navidrome server, the LLM provider, a TTS engine and the DJ's personality, and offers to render jingles. Almost nothing else is configured through Runtipi — the station's settings live in its own state directory and are edited in the admin console at `/admin`.

## Configuration notes

- **Admin username and password** — required; the controller refuses to start without them. They guard `/onboarding`, `/admin` and the admin half of the API.
- **Public URL** — optional, and only used where the station has to print an absolute address: share cards, the sitemap, and the `/listen.pls` / `/listen.m3u` tune-in files. Without it those fall back to whatever address the listener arrived on, which is usually fine on a LAN. Set it (and restart) once you expose the app on a domain.
- **Reaching Navidrome** — the containers get `host.docker.internal`, so a Navidrome on the same host is reachable as `http://host.docker.internal:4533`; a Navidrome elsewhere just takes its LAN address. Navidrome ≥ 0.62 is recommended — SUB/WAVE streams with `format=raw` so transcode limits never throttle the radio, and it will fold Navidrome's `sonicSimilarity` neighbours into track selection when the extension is on.
- **No LLM to hand?** Install Ollama and point the station at a local model, or use any cloud provider's API key. The music keeps playing without a working LLM; only the chatter stops.
- **Voices** — Piper and Kokoro run inside the image and cover most stations. The heavyweight Chatterbox (voice cloning) and PocketTTS engines live in an optional ~6 GB PyTorch sidecar that is **not** part of this build; personas set to them fall back to Piper. The cloud engines and the Remote engine (your own HTTP endpoint) work without it.
- **Acoustic analysis** — runs in-process and handles BPM, key, loudness and endings. The heavier "sounds-like" (CLAP) embeddings and Demucs vocal ranges need upstream's separate heavy image and are not enabled here.
- **System stats panel** — upstream can feed the admin Stats panel through a Docker socket proxy. This build leaves it out, so that one panel stays empty; everything else in the admin console works.
- **Exposing it** — the web UI, the API and the audio stream all come out of one internal Caddy edge on a single port, so a single Runtipi exposure covers the whole station.
- **amd64 only** — this is upstream's all-in-one build, which is published for amd64 alone. There is no arm64 image, so it will not run on a Pi or an Apple-Silicon host.

## Music licensing

SUB/WAVE is playback and automation software and ships with no licensed content. Owning a file covers your own private listening; it does **not** cover **public performance**. The moment the station streams to anyone but you, you are publicly performing copyrighted works, which in most countries needs licences for both the composition (PRS, ASCAP/BMI/SESAC) and the sound recording (PPL, SoundExchange). Keep the station private, or broadcast only material you are cleared to use.

## Data layout

- `${APP_DATA_DIR}/data/state` — everything that matters: settings, the DJ personas, shows, skills, playlists, the library cache (SQLite), rendered voices, jingles and the hourly archives. **Back this directory up and you have backed up the station.**

Source: <https://github.com/perminder-klair/subwave> · Docker image: `ghcr.io/perminder-klair/subwave-aio` — icecast2, Liquidsoap, the DJ controller, the web UI and the Caddy edge in one container, the same build upstream ships for one-container platforms
