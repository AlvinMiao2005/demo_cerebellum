# Cerebellum Cognitive Mapping — Task Battery

A reusable, standardized battery of computerized cognitive assessment tasks developed for research into cerebellar contributions to domain-specific cognitive and motor function. This codebase is a demo excerpt from an active research project at the CognAC (Cognition and Action) Lab, UC Berkeley, contributed as part of the **Raynor Cerebellum Project (RCP)**, a global interdisciplinary initiative studying cerebellar dysfunction and disorders.

The battery integrates classical cognitive paradigms into standardized, pygame-driven behavioral experiments designed for use in clinical research with patients with cerebellar ataxia, and supports downstream VLSM- and fMRI-informed analyses (including drift-diffusion / HSSM modeling of response-time data).

**Role:** The author independently designed and implemented all code in this repository, and led the underlying experimental design and literature review. Beyond delivering the individual task modules, the author engineered a **shared, reusable task-engine template** (`template/`) that every task in the battery is built on — see [Framework Architecture](#framework-architecture) below. Any researcher on the project can extend the battery with a new standardized task by cloning that template and filling in a small, well-defined set of extension points, rather than writing a new experiment runner from scratch.

---

## Environment Setup

### Requirements

- **Python 3.10+** (developed and tested on Python 3.10.12). This floor is a hard requirement, not just a recommendation: session/runtime state throughout the codebase (e.g. `utils/config.py`) is type-annotated with `str | None`-style unions evaluated directly at runtime (PEP 604), which only works natively on Python 3.10+.
- **Package manager:** [Conda](https://docs.conda.io/) (recommended; the project's editor configuration assumes a Conda-managed environment and interpreter)
- **Runtime dependency:** [`pygame`](https://www.pygame.org/) (`>= 2.5`) — the only third-party package needed by the shared template and 9 of the 10 task modules. It handles stimulus rendering, display/window management, and keyboard/joystick input.
- **Additional dependencies for `et` only:** `pandas`, `numpy` and `opencv-python` (video-clip playback and rating-trace handling). All are listed in [`requirements.txt`](./requirements.txt).
- **Input device:** keyboard only is sufficient to run every task. A joystick is auto-detected and used if present (`pygame.joystick`), but every module falls back cleanly to keyboard-only responses when none is connected — no special hardware is required to try the code.

Everything else imported across the codebase, apart from the `et` dependencies above (`dataclasses`, `pathlib`, `logging`, `csv`, `itertools`, `re`, `math`, `datetime`, `typing`, `collections`, `traceback`) is part of the Python standard library.

### Setup

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd demo_cerebellum
   ```

2. **Create and activate a Conda environment**

   ```bash
   conda create -n cerebellum python=3.10
   conda activate cerebellum
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **(Optional) VS Code configuration**

   This repository ships with a `.vscode/settings.json` that registers the project with the [Python Environments extension](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-python-envs), using Conda as both the environment and package manager. If you use VS Code, opening the folder will prompt you to select or create the matching Conda environment automatically.

### Running a task

Each task module is self-contained under its own directory, with a `src/main.py` entry point:

```bash
cd <task_module>
python -m src.main
```

Each task launches full-screen by default and auto-detects the display resolution (or falls back to a fixed default). Press `ESC` at any time to toggle out of fullscreen. Refer to each module's `resources/` directory for stimulus assets and task-specific configuration.

On first run, a task module creates its own `logs/` and `results/` folders (already present as empty, git-tracked placeholders — see Repository Structure) and writes a per-session log file plus a per-participant results CSV.

---

## Repository Structure

```
demo_cerebellum/
├── 2d/               # Task module (built from template/)
├── 3d/               # Task module (built from template/)
├── ccc/              # Task module (built from template/)
├── ccs/              # Task module (built from template/)
├── clamp/            # Task module (built from template/)
├── et/               # Task module (built from template/)
├── gss/              # Task module (built from template/)
├── ied/              # Task module (built from template/)
├── nBack/            # Task module (built from template/)
├── sd/               # Task module (built from template/)
├── template/         # Shared task-engine template — see "Framework Architecture" below
├── .vscode/          # Editor + environment configuration
├── .gitignore
├── requirements.txt  # pygame (+ pandas, numpy, opencv-python for et)
└── README.md
```

Each task module follows the same layout, inherited directly from `template/`:

```
<module>/
├── src/          # Task implementation (entry point: main.py)
├── resources/    # Stimulus assets and configuration files
├── logs/         # Runtime logs (empty, git-tracked via .gitkeep; contents not tracked)
└── results/      # Participant output data (empty, git-tracked via .gitkeep; contents not tracked)
```

`src/` itself always splits into `core/` (task/trial logic — see below), `ui/` (rendering), and `utils/` (shared infrastructure), per the [Framework Architecture](#framework-architecture).

### What's unique to each module

The `ui/` and `utils/` layers are shared template infrastructure (see below); what differs per task is almost entirely contained in `src/core/`:

| Module | `src/core/` contents |
|---|---|
| `2d` / `3d` | `practice.py`, `test.py`, `experiment_flow.py` — mental-rotation-style letter/shape judgment task in 2D / 3D |
| `ccc` | `construct_trials.py`, `single_tasks.py`, `multi_tasks.py`, `experiment_flow.py` — single-task vs. multi-task condition design |
| `ccs` | `framework.py`, `motor.py`, `sensorimotor.py`, `stimuli.py`, `instructions.py`, `feedback.py`, `generate_trials.py`, `experiment_flow.py` — motor and sensorimotor sub-tasks |
| `clamp` | `clamp.py`, `practice.py`, `test.py`, `experiment_flow.py` — visuomotor clamp task with a hidden mouse and cursor-feedback conditions |
| `et` | `experiment_flow.py`, `mouseCordDot.py`, `practice.py`, `stimulus.py`, `saves.py`, `test.py` — emotion tracking: continuous trackpad rating of a film character's emotion during video clips |
| `gss` | `basic_practice.py`, `goal_practice.py`, `test.py`, `experiment_flow.py` — goal-directed (speed/accuracy) Stroop-style task with DDM-ready trial data |
| `ied` | `practice.py`, `test.py`, `experiment_flow.py` |
| `nBack` | `one_back.py`, `two_back.py`, `three_back.py`, `pull_stimuli.py`, `experiment_flow.py` — classic N-back working-memory task |
| `sd` | `practice.py`, `test.py`, `main_window.py` (`saves.py` lives in `core/` for this module) — sentence-based semantic/cloze-probability decision task |
| `template` | `practice.py`, `test.py`, `experiment_flow.py` — generic 2-option stimulus-response scaffold |

---

## Framework Architecture

`template/` is not sample code — it is the shared engine that powers every task in the battery. Each of the ten task modules above began as a copy of `template/`, then customized its stimuli, response mapping, and trial/save logic (the `src/core/` contents in the table above). This gives the whole battery a common runtime, a common results schema, and a common participant-facing flow, so data collected across different cognitive paradigms stays directly comparable and a new task can be stood up in hours rather than days.

### Execution flow

```
main.py
  └─ core/experiment_flow.py   (run)         orchestrates the full session
       ├─ ui/pygame_render.py                display init, fullscreen toggle,
       │                                     participant ID + admin capture
       │                                     (group / session / dominant hand /
       │                                     hand used), instruction pages
       ├─ core/practice.py     (run_practice) practice block, with trial feedback
       └─ core/test.py         (run_test)     experimental block(s), no feedback
```

### Shared infrastructure (`src/utils/`)

| Module | Responsibility |
|---|---|
| `config.py` | Single source of truth for runtime parameters (timing, screen, colors, joystick deadzone/mode) and session state (PID, mapping, group, session, hand). A `MODE = "demo" / "full"` switch scales timing constants for fast iteration during development vs. real participant runs. |
| `paths.py` | Resolves all asset paths (instructions, stimuli, feedback, admin screens) relative to the project root, including counterbalanced-mapping variants (`instructions_v1` / `instructions_v2`) and dynamic binding of admin images by filename. |
| `event_handler.py` | Normalizes raw pygame keyboard and joystick events into a single frame-level `ControlState`, so trial logic never touches pygame's event API directly. Supports both 2-direction and 4-direction joystick response schemes. |
| `logger.py` | Centralized file + console logger factory shared by every module. |
| `saves.py` | Creates and appends to the participant's per-session results CSV with a fixed column schema, automatic per-participant/per-day file naming and collision handling, and key/joystick response routing. |

### Extension points for a new task

Adapting the template to a new cognitive paradigm touches a bounded, well-documented set of files (tracked in the template's own `TODO.md`):

- `core/test.py` / `core/practice.py` — block/trial design and which fields get saved per trial
- `core/experiment_flow.py` — overall task flow (instruction pages, block order)
- `utils/config.py` — task-specific parameters (timing, joystick mode, demo vs. full mode)
- `utils/event_handler.py` — semantic meaning of response options for the new task
- `utils/paths.py` — additional instruction/stimulus/feedback asset paths
- `utils/saves.py` / `utils/logger.py` — task name and any additional result columns

Everything else — display setup, fullscreen handling, participant/admin intake, image rendering and scaling, feedback presentation, and CSV persistence — is inherited unchanged, so each new task module stays consistent with the rest of the battery by construction rather than by convention.

---

## Notes

- Generated runtime artifacts (`logs/*`, `results/*`, `__pycache__/`, virtual environments) are excluded from version control via `.gitignore`; the `logs/` and `results/` folders themselves are kept in git via `.gitkeep` placeholders so the expected output locations are visible without running anything.
- **`et` assets:** film stills and video clips are not redistributable and are omitted. Baseline frames and instruction/practice screenshots are generated placeholders, `resources/stimuli/**/*.mp4` is git-ignored, and the task needs user-supplied clips to run end to end.
- This repository is a portfolio demo derived from an active lab codebase; internal experimental notes and participant data have been excluded — nothing under `resources/` is real participant data, and no `logs/` or `results/` output from real sessions is included.
- No license file is included yet; this repository is shared as a code-sample / portfolio excerpt rather than a redistributable package. Add a `LICENSE` if you intend to make it reusable by others under specific terms.
