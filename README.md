# STEMViz — AI STEM Course Generator

**AI-powered STEM concept visualizer that generates narrated educational animations using Manim, LLMs, and multimodal AI.**

Transform any STEM concept into a narrated video animation with a single text prompt. STEMViz runs a multi-agent pipeline that interprets your concept, generates Manim animation code, renders scenes in parallel, and synchronizes AI-generated narration with the final video.

---

## Features

- **Multi-Agent Architecture**: Specialized agents for concept decomposition (Claude) and Manim code generation
- **Parallel Scene Rendering**: Concurrent scene code generation via `ThreadPoolExecutor`
- **Multimodal Narration**: Gemini 2.5 Flash analyzes the silent animation and generates timestamped narration
- **Pluggable TTS**: Swap between ElevenLabs and OpenAI TTS via a single env variable
- **Multi-Language Support**: Generate narration in English, Chinese, Spanish, or Vietnamese
- **Gradio Web Interface**: Browser-based UI — no CLI required
- **Auto Cleanup**: Temporary files removed after successful generation; only final video and SRT are kept

---

## Pipeline Overview

```
User Input (STEM concept + language)
  ↓
Concept Interpreter Agent          [Claude Sonnet 4.5 via OpenRouter]
  → Structured sub-concept analysis with dependencies
  ↓
Manim Agent                        [Claude Sonnet 4.5 via OpenRouter]
  → Scene planning → parallel code generation → Manim rendering
  ↓
Silent Animation (concatenated MP4)
  ↓
Script Generator                   [Gemini 2.5 Flash — multimodal]
  → Analyzes video frames → timestamped SRT narration
  ↓
TTS Synthesizer                    [ElevenLabs or OpenAI]
  → Audio segments → padded + concatenated audio track
  ↓
Video Compositor                   [FFmpeg]
  → Final MP4 with audio + burned-in subtitles
  ↓
Output in output/final/
```

---

## Installation

### Prerequisites

**System Requirements:**
- Python 3.10+
- FFmpeg (for video processing)
- LaTeX (for mathematical notation in Manim animations)

**API Keys Required:**
- [OpenRouter API Key](https://openrouter.ai/) — LLM reasoning (Claude)
- [Google AI API Key](https://aistudio.google.com/app/apikey) — multimodal video analysis (Gemini)
- **One of:**
  - [ElevenLabs API Key](https://elevenlabs.io/) (default TTS provider)
  - [OpenAI API Key](https://platform.openai.com/) (alternative TTS provider)

---

### Step 1: Install System Dependencies

#### macOS
```bash
brew install ffmpeg
brew install --cask mactex

# Add LaTeX to PATH (add to ~/.zshrc or ~/.bash_profile)
export PATH="/Library/TeX/texbin:$PATH"
source ~/.zshrc
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install ffmpeg texlive-full
```

#### Windows
```powershell
# Run PowerShell as Administrator
choco install ffmpeg miktex
```

**Verify:**
```bash
ffmpeg -version
latex --version
```

---

### Step 2: Clone Repository

```bash
git clone https://github.com/MinhTuanDang/STEMViz.git
cd STEMViz
```

---

### Step 3: Install Python Dependencies

We recommend [UV](https://github.com/astral-sh/uv):

**macOS/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

```bash
uv venv
source .venv/bin/activate        # macOS/Linux
# .venv\Scripts\activate         # Windows

uv pip install -r requirements.txt
```

**Alternative (pip):**
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

### Step 4: Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env`:

```bash
# ── Required ────────────────────────────────────────────
OPENROUTER_API_KEY=your_openrouter_key
GOOGLE_API_KEY=your_google_ai_key

# ── TTS Provider ────────────────────────────────────────
TTS_PROVIDER=elevenlabs            # or "openai"

# ElevenLabs (required if TTS_PROVIDER=elevenlabs)
ELEVENLABS_API_KEY=your_elevenlabs_key

# OpenAI (required if TTS_PROVIDER=openai)
OPENAI_API_KEY=your_openai_key
OPENAI_ENDPOINT=                   # Optional: custom-compatible endpoint (e.g. Kokoro)

# ── Optional Overrides ──────────────────────────────────
MANIM_QUALITY=p                    # l=480p15  m=720p30  h=1080p30  p=1080p60  k=1440p60
TARGET_LANGUAGE=English            # English | Chinese | Spanish | Vietnamese
MANIM_BACKGROUND_COLOR=#0f0f0f
SUBTITLE_POSITION=bottom           # top | center | bottom
SUBTITLE_BACKGROUND_OPACITY=0.5
VIDEO_PRESET=medium                # ultrafast → veryslow
VIDEO_CRF=23                       # 0–51 (lower = better quality)
```

---

### Step 5: Verify Manim

```bash
manim --version
```

If not found, reinstall:
```bash
uv pip install --force-reinstall manim
```

---

## Usage

### Launch Web Interface

```bash
python app.py
```

Opens at `http://127.0.0.1:7860`

### Using the Interface

1. **Enter a STEM concept** in the text box
2. **Select language** (English, Chinese, Spanish, Vietnamese)
3. **Click "Generate Animation"**
4. **Wait ~3–5 minutes** depending on concept complexity
5. **Watch the generated video** with synchronized narration

### Example Prompts

```
Explain bubble sort algorithm
Demonstrate gradient descent optimization
Show Bayes' theorem with a medical diagnosis example
Explain how backpropagation works in neural networks
Visualize the Fourier transform
Demonstrate the central limit theorem
```

---

## Configuration Reference

All settings live in `config.py` and can be overridden via `.env`.

### LLM Settings

| Variable | Default | Description |
|---|---|---|
| `REASONING_MODEL` | `anthropic/claude-sonnet-4.5` | OpenRouter model for agents |
| `MULTIMODAL_MODEL` | `gemini-2.5-pro` | Gemini model for narration |
| `INTERPRETER_REASONING_TOKENS` | `2048` | Thinking budget for concept agent |
| `ANIMATION_REASONING_TOKENS` | `4096` | Thinking budget for Manim agent |
| `INTERPRETER_REASONING_EFFORT` | `low` | Reasoning effort (low/medium/high) |
| `ANIMATION_REASONING_EFFORT` | `medium` | Reasoning effort (low/medium/high) |
| `ANIMATION_TEMPERATURE` | `0.5` | Manim code generation temperature |
| `SCRIPT_GENERATION_TEMPERATURE` | `0.5` | Narration generation temperature |

### Animation Settings

| Variable | Default | Description |
|---|---|---|
| `MANIM_QUALITY` | `p` | Render quality (l/m/h/p/k) |
| `MANIM_MAX_SCENE_DURATION` | `30.0` | Max seconds per scene |
| `MANIM_TOTAL_VIDEO_DURATION_TARGET` | `120.0` | Target final video length (seconds) |
| `MANIM_RENDER_TIMEOUT` | `300` | Subprocess timeout per scene (seconds) |
| `MANIM_BACKGROUND_COLOR` | `#0f0f0f` | Scene background hex color |

### TTS Settings

#### ElevenLabs
| Variable | Default | Description |
|---|---|---|
| `ELEVENLABS_VOICE_ID` | `JBFqnCBsd6RMkjVDRZzb` | Voice ID |
| `ELEVENLABS_MODEL_ID` | `eleven_multilingual_v2` | Model |
| `ELEVENLABS_STABILITY` | `0.75` | Voice stability (0–1) |
| `ELEVENLABS_SIMILARITY_BOOST` | `0.75` | Clarity boost (0–1) |
| `ELEVENLABS_STYLE` | `0.0` | Style exaggeration (0–1) |
| `ELEVENLABS_USE_SPEAKER_BOOST` | `true` | Speaker boost |

#### OpenAI
| Variable | Default | Description |
|---|---|---|
| `OPENAI_VOICE` | `alloy` | Voice (alloy/echo/fable/onyx/nova/shimmer) |
| `OPENAI_MODEL` | `tts-1` | Model (tts-1 or tts-1-hd) |
| `OPENAI_SPEED` | `1.0` | Playback speed (0.25–4.0) |
| `OPENAI_RESPONSE_FORMAT` | `mp3` | Audio format |

### Video Settings

| Variable | Default | Description |
|---|---|---|
| `VIDEO_CODEC` | `libx264` | FFmpeg video codec |
| `VIDEO_CRF` | `23` | Constant rate factor (0–51) |
| `VIDEO_PRESET` | `medium` | Encoding speed/compression tradeoff |
| `AUDIO_CODEC` | `aac` | FFmpeg audio codec |
| `AUDIO_BITRATE` | `128k` | Audio bitrate |
| `SUBTITLE_BURN_IN` | `true` | Hard-code subtitles into video |
| `SUBTITLE_POSITION` | `bottom` | Subtitle position (top/center/bottom) |
| `SUBTITLE_BACKGROUND_OPACITY` | `0.5` | Subtitle box opacity |

---

## Output Structure

```
output/
├── analyses/       # Concept analysis JSON
├── scene_codes/    # Generated Manim .py files     [cleaned on success]
├── scenes/         # Rendered scene MP4s           [cleaned on success]
├── animations/     # Concatenated silent animation [cleaned on success]
├── scripts/        # SRT narration files           [KEPT]
├── audio/          # Final padded audio track      [cleaned on success]
│   └── segments/   # Per-subtitle audio clips      [cleaned on success]
└── final/          # Final video with narration    [KEPT]

output/pipeline.log # Full run log
```

---

## Architecture

### Project Structure

```
STEMViz/
├── agents/
│   ├── base.py                   # BaseAgent — OpenRouter REST client, retry logic
│   ├── concept_interpreter.py    # Decomposes concept into ordered sub-concepts
│   ├── manim_agent.py            # Plans scenes, generates + renders Manim code
│   └── manim_models.py           # Pydantic models for animation pipeline
├── generation/
│   ├── script_generator.py       # Gemini multimodal → timestamped SRT
│   ├── video_compositor.py       # FFmpeg video + audio + subtitle composition
│   └── tts/
│       ├── base.py               # BaseTTSSynthesizer — SRT parsing, audio concat
│       ├── elevenlabs_provider.py
│       └── openai_provider.py
├── rendering/
│   └── manim_renderer.py         # Subprocess-isolated Manim execution
├── utils/
│   └── validators.py             # Input/output validation, FFmpeg checks
├── config.py                     # Pydantic Settings — all configuration
├── pipeline.py                   # Orchestration engine
├── app.py                        # Gradio web interface
└── TTS_USAGE.md                  # TTS provider guide
```

### Technology Stack

| Component | Technology |
|---|---|
| Web UI | Gradio |
| Animation engine | Manim Community Edition |
| Concept & code agent | Claude Sonnet 4.5 (OpenRouter) |
| Narration analysis | Gemini 2.5 Flash |
| TTS (default) | ElevenLabs eleven_multilingual_v2 |
| TTS (alternative) | OpenAI tts-1 / tts-1-hd |
| Media processing | FFmpeg |
| Config validation | Pydantic Settings |

---

## TTS Providers

See [`TTS_USAGE.md`](TTS_USAGE.md) for the full guide including programmatic usage and how to add custom providers.

**Quick switch:**
```bash
# ElevenLabs (default)
TTS_PROVIDER=elevenlabs
ELEVENLABS_API_KEY=...

# OpenAI
TTS_PROVIDER=openai
OPENAI_API_KEY=...

# OpenAI-compatible endpoint (e.g., self-hosted Kokoro)
TTS_PROVIDER=openai
OPENAI_API_KEY=dummy
OPENAI_ENDPOINT=http://localhost:8880
```

---

## Troubleshooting

### "LaTeX not found"
```bash
latex --version

# macOS — add to PATH:
export PATH="/Library/TeX/texbin:$PATH"

# Linux — reinstall:
sudo apt install texlive-full
```

### "FFmpeg not found"
```bash
ffmpeg -version
brew reinstall ffmpeg     # macOS
sudo apt reinstall ffmpeg # Linux
```

### "Manim command not found"
```bash
source .venv/bin/activate
uv pip install --force-reinstall manim
```

### API key errors
- Confirm `.env` exists and keys have no extra spaces or quotes
- Verify quotas on [OpenRouter](https://openrouter.ai/), [Google AI Studio](https://aistudio.google.com/), and your TTS provider

### Out of memory / slow rendering
- Lower quality: `MANIM_QUALITY=m` (720p30) or `MANIM_QUALITY=l` (480p15)
- Reduce `MANIM_MAX_SCENE_DURATION` in `config.py`
- First run downloads LaTeX packages — subsequent runs are faster

### Checking logs
```bash
cat output/pipeline.log
```

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push: `git push origin feature/your-feature`
5. Open a Pull Request

---

## Acknowledgments

- **Manim Community** — mathematical animation engine
- **3Blue1Brown** — inspiration for educational math visualizations
- **Anthropic, Google, ElevenLabs, OpenAI** — AI APIs powering the pipeline

---

**Star this repo if you find it useful!**
