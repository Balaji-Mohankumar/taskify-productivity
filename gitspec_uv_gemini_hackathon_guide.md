# GitHub Spec Kit ("GitSpec") + UV + Gemini Pro: Complete Single-Folder Hackathon Guide

## 1. What is GitHub Spec Kit ("GitSpec") & How Does It Work?

**GitHub Spec Kit** ([github/spec-kit](https://github.com/github/spec-kit), often referred to as **GitSpec** or **Specify CLI**) is an open-source **Specification-Driven Development (SDD)** toolkit created by GitHub.

During a hackathon, jumping straight into "vibe coding" (prompting an LLM to write code directly from a vague idea) quickly leads to broken dependencies, hallucinated APIs, and messy architecture. **Spec Kit flips that script**: instead of treating code as the primary artifact and markdown prompts as throwaway scratchpads, **your specifications become the executable source of truth**.

### What Lies Inside a Spec Kit Project? (The Folder Anatomy)

When you initialize Spec Kit inside your workspace folder, it generates three core directories that drive your LLM:

| Path | What It Contains | How Your Idea Uses It |
| :--- | :--- | :--- |
| `.specify/memory/constitution.md` | **The Project Constitution**: Your non-negotiable architectural rules, tech stack choices, coding standards, and hackathon constraints (e.g., "Use Python 3.12, FastAPI, SQLite, zero external cloud DBs"). | Every time Gemini generates a plan or writes code, it reads `constitution.md` first so it never violates your stack or invents unwanted frameworks. |
| `.specify/templates/` & `.specify/scripts/` | **Templates & Automation Scripts**: Structured Markdown blueprints (`spec-template.md`, `plan-template.md`, `tasks-template.md`) and helper scripts (`create-new-feature.sh` / `.ps1`) that enforce strict formatting. | Keeps your LLM outputs deterministic. When you ask for a feature spec or implementation plan, the templates force the LLM to separate *What/Why* from *How*. |
| `.gemini/commands/speckit.*.toml` | **Agent Slash Commands**: Custom TOML command definitions (or `.github/prompts/` for VS Code agents) that register `/speckit.*` commands directly into your AI coding assistant. | Gives you a repeatable 5-step command pipeline (`/speckit.constitution` → `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`). |
| `specs/<NNN-feature-name>/` | **Feature Artifacts**: Each feature or hackathon module gets its own numbered folder (e.g., `specs/001-core-mvp/`) containing `spec.md`, `plan.md`, `data-model.md`, and `tasks.md`. | Turns your raw hackathon idea into a concrete PRD (`spec.md`), technical architecture (`plan.md`), and an ordered checklist of coding tasks (`tasks.md`) before a single line of code is written. |

---

## 2. Confinement Architecture: Everything in One Folder

By default, `uv` installs binaries into `~/.local/bin`, Python versions into `~/.local/share/uv/python`, and caches into `~/.cache/uv`. To keep **100% of your installation—including the `uv` binary, Python 3.12, `specify-cli`, virtual environments, and API keys—strictly inside a single project folder** (e.g., `hackathon-workspace/`), we override `uv`'s storage paths using local environment variables:

```text
hackathon-workspace/                  <-- Your Single Self-Contained Root Folder
├── .local-runtime/                   <-- Confined Tooling & Python Runtime (Never touches global OS)
│   ├── bin/                          <-- uv, uvx, and specify binaries live here
│   ├── python/                       <-- Isolated Python 3.12 installation lives here
│   ├── tools/                        <-- Isolated specify-cli tool environment
│   └── cache/                        <-- Local uv package cache
├── .venv/                            <-- Project-specific Python virtual environment
├── .vscode/
│   └── settings.json                 <-- Automatically injects local env vars & paths into VS Code terminals
├── .env                              <-- Local API keys (GEMINI_API_KEY) & local UV directory overrides
├── .specify/                         <-- Spec Kit engine (memory/constitution.md, templates, scripts)
├── .gemini/                          <-- Gemini CLI slash commands (.gemini/commands/speckit.*.toml)
├── specs/                            <-- Your feature specifications, plans, and task breakdowns
└── src/                              <-- Your actual Python application code
```

---

## 3. Step-by-Step Setup: Local UV, Python, and Spec Kit

Open **VS Code**, open a terminal, and follow these steps for your operating system to build the self-contained folder.

### Step 3.1: Create Your Single Workspace Folder

**macOS / Linux (Bash / Zsh):**
```bash
mkdir -p hackathon-workspace/.local-runtime/{bin,python,tools,cache}
cd hackathon-workspace
code .
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "hackathon-workspace\.local-runtime\bin", "hackathon-workspace\.local-runtime\python", "hackathon-workspace\.local-runtime\tools", "hackathon-workspace\.local-runtime\cache"
Set-Location hackathon-workspace
code .
```

### Step 3.2: Install `uv` Strictly Inside `.local-runtime/bin`

We use `UV_INSTALL_DIR` and `INSTALLER_NO_MODIFY_PATH=1` so the installer drops the `uv` binary into your folder without touching your global shell profile.

**macOS / Linux:**
```bash
export WORKSPACE_ROOT="$(pwd)"
export UV_INSTALL_DIR="$WORKSPACE_ROOT/.local-runtime/bin"
export UV_PYTHON_INSTALL_DIR="$WORKSPACE_ROOT/.local-runtime/python"
export UV_TOOL_DIR="$WORKSPACE_ROOT/.local-runtime/tools"
export UV_TOOL_BIN_DIR="$WORKSPACE_ROOT/.local-runtime/bin"
export UV_CACHE_DIR="$WORKSPACE_ROOT/.local-runtime/cache"
export PATH="$WORKSPACE_ROOT/.local-runtime/bin:$PATH"

curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR="$UV_INSTALL_DIR" INSTALLER_NO_MODIFY_PATH=1 sh
```

**Windows (PowerShell):**
```powershell
$env:WORKSPACE_ROOT = (Get-Location).Path
$env:UV_INSTALL_DIR = "$env:WORKSPACE_ROOT\.local-runtime\bin"
$env:UV_PYTHON_INSTALL_DIR = "$env:WORKSPACE_ROOT\.local-runtime\python"
$env:UV_TOOL_DIR = "$env:WORKSPACE_ROOT\.local-runtime\tools"
$env:UV_TOOL_BIN_DIR = "$env:WORKSPACE_ROOT\.local-runtime\bin"
$env:UV_CACHE_DIR = "$env:WORKSPACE_ROOT\.local-runtime\cache"
$env:PATH = "$env:WORKSPACE_ROOT\.local-runtime\bin;$env:PATH"

$env:UV_NO_MODIFY_PATH = "1"
irm https://astral.sh/uv/install.ps1 | iex
```

Verify that `uv` is running from your local folder:
```bash
uv --version
```

### Step 3.3: Install Python 3.12 & Create a Local Virtual Environment

Now instruct `uv` to download an isolated Python 3.12 runtime into `.local-runtime/python` and create a `.venv` inside your workspace:

```bash
uv python install 3.12
uv venv --python 3.12 .venv
```

### Step 3.4: Install GitHub Spec Kit (`specify-cli`) Locally

Install `specify-cli` into `.local-runtime/tools` and expose the `specify` executable inside `.local-runtime/bin`:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
specify --version
```

---

## 4. Local Environment Variables & VS Code Auto-Configuration

To avoid typing `export` commands every time you open VS Code, create two config files inside `hackathon-workspace/`: a `.env` file and `.vscode/settings.json`.

### Step 4.1: Create Your Local `.env` File

Create a file named `.env` in the root of `hackathon-workspace/`:

```ini
# Local UV Confinement Paths
UV_PYTHON_INSTALL_DIR=./.local-runtime/python
UV_TOOL_DIR=./.local-runtime/tools
UV_TOOL_BIN_DIR=./.local-runtime/bin
UV_CACHE_DIR=./.local-runtime/cache

# Gemini API Configuration (Get a key from https://aistudio.google.com/apikey if using API mode)
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-2.5-pro
```

*(Note: Add `.local-runtime/`, `.venv/`, and `.env` to your `.gitignore` so you never commit binaries or secrets to Git.)*

### Step 4.2: Configure `.vscode/settings.json` for Automatic Terminal Isolation

Create `.vscode/settings.json` inside `hackathon-workspace/`. Whenever you open a new terminal in VS Code, it will automatically activate `.venv`, prepend `.local-runtime/bin` to your `PATH`, and load `.env`:

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true,
  "python.envFile": "${workspaceFolder}/.env",
  "terminal.integrated.env.osx": {
    "UV_PYTHON_INSTALL_DIR": "${workspaceFolder}/.local-runtime/python",
    "UV_TOOL_DIR": "${workspaceFolder}/.local-runtime/tools",
    "UV_TOOL_BIN_DIR": "${workspaceFolder}/.local-runtime/bin",
    "UV_CACHE_DIR": "${workspaceFolder}/.local-runtime/cache",
    "PATH": "${workspaceFolder}/.local-runtime/bin:${workspaceFolder}/.venv/bin:${env:PATH}"
  },
  "terminal.integrated.env.linux": {
    "UV_PYTHON_INSTALL_DIR": "${workspaceFolder}/.local-runtime/python",
    "UV_TOOL_DIR": "${workspaceFolder}/.local-runtime/tools",
    "UV_TOOL_BIN_DIR": "${workspaceFolder}/.local-runtime/bin",
    "UV_CACHE_DIR": "${workspaceFolder}/.local-runtime/cache",
    "PATH": "${workspaceFolder}/.local-runtime/bin:${workspaceFolder}/.venv/bin:${env:PATH}"
  },
  "terminal.integrated.env.windows": {
    "UV_PYTHON_INSTALL_DIR": "${workspaceFolder}\\.local-runtime\\python",
    "UV_TOOL_DIR": "${workspaceFolder}\\.local-runtime\\tools",
    "UV_TOOL_BIN_DIR": "${workspaceFolder}\\.local-runtime\\bin",
    "UV_CACHE_DIR": "${workspaceFolder}\\.local-runtime\\cache",
    "PATH": "${workspaceFolder}\\.local-runtime\\bin;${workspaceFolder}\\.venv\\Scripts;${env:PATH}"
  }
}
```

---

## 5. Connecting Your Gemini Pro Account to Spec Kit

You can use your **Gemini Pro account** with Spec Kit in two ways inside VS Code:

1. **Via Official Gemini CLI (`--integration gemini`)**:
   - Install Google's open-source Gemini CLI ([google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)) or run it via `npx @google/gemini-cli`.
   - When you launch `gemini` in your VS Code terminal, choose **"Login with Google"** to authenticate directly with your Gemini Pro account (or select **"Use Gemini API key"** to use `GEMINI_API_KEY` from your `.env` file).
2. **Via Google AI SDK in Python (`google-genai`)**:
   - If your hackathon project itself calls Gemini models (e.g., building an AI agent or analyzer), install the official SDK inside your local `.venv`:
     ```bash
     uv pip install google-genai python-dotenv rich
     ```

### Initialize Spec Kit in Your Workspace

Run `specify init` inside `hackathon-workspace/` with the `gemini` integration flag:

```bash
specify init --here --integration gemini
```

What this command just created in your folder:
- `.specify/memory/constitution.md` — Template for your project's architectural rules.
- `.specify/templates/` — Structured templates for specs, plans, and tasks.
- `.gemini/commands/` — All `/speckit.*` slash commands formatted for Gemini (`speckit.constitution.toml`, `speckit.specify.toml`, `speckit.plan.toml`, `speckit.tasks.toml`, `speckit.implement.toml`).

---

## 6. Hands-On Tutorial: Build a Small, Meaningful Project Step-by-Step

To get comfortable with the workflow before your hackathon on Monday, let's build **"PitchCritic"**—a CLI tool that reads a project README or pitch markdown file, evaluates it against hackathon judging criteria using Gemini (`google-genai`), and outputs a structured readiness scorecard.

### Step 1: Establish the Project Constitution (`/speckit.constitution`)

The Constitution defines **how** code should be written in your repository. In your Gemini CLI chat (or by editing `.specify/memory/constitution.md` directly with your LLM), run:

```text
/speckit.constitution Create principles for a rapid Python 3.12 hackathon project managed with uv.
1. Dependency discipline: Use only standard library, google-genai, python-dotenv, and rich.
2. Configuration: Load all secrets and model names strictly from the local .env file.
3. Simplicity: Prefer a clean, modular single-package layout under src/pitch_critic/ with type hints and clear error handling.
4. Verification: Every feature must include a runnable smoke test using pytest.
```

**Check the output**: Open `.specify/memory/constitution.md` in VS Code. You will see these rules codified at the top of your project memory.

### Step 2: Write the Feature Specification (`/speckit.specify`)

Next, describe **what** you want to build and **why**—without worrying about code details yet. Run:

```text
/speckit.specify Build a CLI tool called PitchCritic that takes a path to a markdown file (like a hackathon idea or README.md) and evaluates it across 4 criteria: Problem Clarity, Technical Feasibility, Innovation, and Demo Potential. It should output a formatted terminal scorecard with scores from 1-10 and 3 concrete action items to improve the pitch before judging.
```

**Check the output**: Spec Kit automatically creates a new branch/folder `specs/001-pitch-critic/spec.md` containing:
- User stories & acceptance criteria
- Functional requirements
- Edge cases (e.g., missing file, empty markdown, missing API key)

*(Optional Pro-Tip)*: Run `/speckit.clarify` right after `/speckit.specify`—Gemini will scan `spec.md` for any ambiguities and ask you targeted multiple-choice questions to lock down edge cases before planning!

### Step 3: Generate the Technical Implementation Plan (`/speckit.plan`)

Now tell Spec Kit **how** to build the spec within your Constitution's rules:

```text
/speckit.plan Use Python 3.12, the google-genai SDK with structured JSON outputs (Pydantic schema) to guarantee reliable scoring, python-dotenv to load the local .env file, and rich.table to render the terminal scorecard.
```

**Check the output**: Look inside `specs/001-pitch-critic/`. Spec Kit generates:
- `plan.md` — Architecture overview, module structure, and library choices verified against `constitution.md`.
- `data-model.md` — The Pydantic schemas (`CriterionScore`, `PitchEvaluation`) that Gemini will populate.

### Step 4: Break the Plan into Executable Tasks (`/speckit.tasks`)

Convert the architecture plan into an ordered, dependency-aware checklist:

```text
/speckit.tasks
```

**Check the output**: Open `specs/001-pitch-critic/tasks.md`. You will see numbered tasks grouped by phase:
- `[T001]` Initialize `pyproject.toml` and install `google-genai`, `pydantic`, `python-dotenv`, `rich`, `pytest` via `uv`.
- `[T002]` Create Pydantic response schemas in `src/pitch_critic/models.py`.
- `[T003]` Implement Gemini API evaluator in `src/pitch_critic/evaluator.py` using `dotenv`.
- `[T004]` Build CLI entry point and Rich table renderer in `src/pitch_critic/cli.py`.
- `[T005]` Add unit test in `tests/test_models.py`.

### Step 5: Execute and Build (`/speckit.implement`)

Now let the agent execute the tasks step-by-step:

```text
/speckit.implement
```

For reference, here is the clean, minimal code structure that `/speckit.implement` produces in your workspace:

**`src/pitch_critic/models.py`:**
```python
from pydantic import BaseModel, Field

class CriterionScore(BaseModel):
    criterion: str
    score: int = Field(ge=1, le=10)
    feedback: str

class PitchEvaluation(BaseModel):
    project_title: str
    overall_readiness: int = Field(ge=1, le=100)
    criteria: list[CriterionScore]
    top_3_improvements: list[str]
```

**`src/pitch_critic/cli.py`:**
```python
import os
import sys
from pathlib import Path
from dotenv import load_dotenv
from google import genai
from google.genai import types
from rich.console import Console
from rich.table import Table
from .models import PitchEvaluation

def evaluate_pitch(md_path: str) -> None:
    # Load strictly from the local folder's .env
    load_dotenv(dotenv_path=Path(__file__).resolve().parents[2] / ".env")
    api_key = os.getenv("GEMINI_API_KEY")
    model_name = os.getenv("GEMINI_MODEL", "gemini-2.5-pro")

    if not api_key or api_key == "your_gemini_api_key_here":
        raise RuntimeError("Set GEMINI_API_KEY in your local .env file first.")

    content = Path(md_path).read_text(encoding="utf-8")
    client = genai.Client(api_key=api_key)

    prompt = (
        "Evaluate the following hackathon project pitch rigorously across: "
        "Problem Clarity, Technical Feasibility, Innovation, and Demo Potential.\n\n"
        f"{content}"
    )

    response = client.models.generate_content(
        model=model_name,
        contents=prompt,
        config=types.GenerateContentConfig(
            response_mime_type="application/json",
            response_schema=PitchEvaluation,
            temperature=0.2,
        ),
    )

    result = PitchEvaluation.model_validate_json(response.text)
    console = Console()
    console.print(f"\n[bold cyan]Hackathon Readiness Scorecard: {result.project_title}[/bold cyan] ({result.overall_readiness}/100)\n")

    table = Table(show_header=True, header_style="bold magenta")
    table.add_column("Criterion", style="bold")
    table.add_column("Score", justify="right")
    table.add_column("Feedback")

    for item in result.criteria:
        table.add_row(item.criterion, f"{item.score}/10", item.feedback)

    console.print(table)
    console.print("\n[bold yellow]Top 3 Action Items Before Judging:[/bold yellow]")
    for idx, action in enumerate(result.top_3_improvements, 1):
        console.print(f"  {idx}. {action}")

if __name__ == "__main__":
    evaluate_pitch(sys.argv[1])
```

Run it inside your confined environment:
```bash
uv run python -m src.pitch_critic.cli specs/001-pitch-critic/spec.md
```

---

## 7. How to Apply Spec Kit to Your Own Hackathon Idea (Monday & Tuesday Playbook)

When you sit down for your hackathon on Monday morning, use this exact strategy to turn **your own idea** into working software without getting derailed:

1. **Before Writing Code (Monday Morning — 30 Mins)**:
   - Open your `hackathon-workspace` folder so `uv` and Python are already isolated and ready.
   - Run `specify extension add assess` if you want to stress-test your idea first (`/speckit.assess.intake "<your idea>"`), **or** jump straight into `/speckit.constitution` to lock in your exact tech stack (e.g., FastAPI + React, Streamlit, or Next.js + Python backend) and forbid the AI from over-engineering (e.g., *"No Docker, no Kubernetes, no complex auth—prioritize a working local demo"*).
2. **Slice Your Idea into 2–3 Feature Specs (Monday Midday)**:
   - Don't dump your entire hackathon project into a single massive prompt.
   - **Feature 001 (`specs/001-core-pipeline/`)**: The core backend/AI pipeline that proves your main value proposition. Run `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`.
   - **Feature 002 (`specs/002-interactive-ui/`)**: The user interface or dashboard built on top of Feature 001.
   - **Feature 003 (`specs/003-demo-polish/`)**: Pre-loaded sample data, fallback responses (in case Wi-Fi drops during live judging), and visual polish.
3. **When Bugs Hit on Tuesday (`/speckit.bug`)**:
   - Install the bundled bug-fixing extension (`specify extension add bug`) and run `/speckit.bug.assess "description of bug" slug=fix-name` so the agent diagnoses the root cause against your spec instead of blindly rewriting working files.
4. **Verify Convergence (`/speckit.converge` or `/speckit.analyze`)**:
   - Run `/speckit.analyze` to check that your implementation matches `spec.md` and `tasks.md` with zero missing requirements before your final presentation.
