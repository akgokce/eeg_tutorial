# PYTHON.MD — Installing Python, Managing Environments, and Conda Basics

Through this guide, we will walk step‑by‑step through installing Python, creating isolated environments, and using common tools that help you manage dependencies cleanly. We will briefly compare common **environment/package managers** (pipx, conda, poetry, uv), and then walk through a practical **conda** workflow (install, create envs, install packages, and export/recreate environments with YAML).

---

## Table of Contents
- [1) Install Python (standard way) & create a venv](#1-install-python-standard-way--create-a-venv)
- [2) Environment/package managers at a glance](#2-environmentpackage-managers-at-a-glance)
- [3) Conda: install, create envs, install packages, export/recreate YAML](#3-conda-install-create-envs-install-packages-exportrecreate-yaml)
  - [3.1 Install Conda](#31-install-conda)
  - [3.2 Configure channels](#32-configure-channels)
  - [3.3 Create & activate an environment](#33-create--activate-an-environment)
  - [3.4 Install packages](#34-install-packages)
  - [3.5 Export your environment to YAML](#35-export-your-environment-to-yaml)
  - [3.6 Create an environment from a YAML file](#36-create-an-environment-from-a-yaml-file)
  - [3.7 Update an existing environment from YAML](#37-update-an-existing-environment-from-yaml)
- [4) Best practices checklist](#4-best-practices-checklist)

---

## 1) Install Python (standard way) & create a `venv`

### Install Python
- **macOS:** Use the official installer or Homebrew (`brew install python`).  
- **Linux:** Use your distro’s package manager (e.g., `sudo apt-get install python3 python3-venv`) or the official installer.

Verify installation:
```bash
python3 --version
```

### Create & use a virtual environment (built-in `venv`)
Create a per-project environment so dependencies don’t clash.

**macOS/Linux:**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Deactivate when done:
```bash
deactivate
```

> **Tip:** Add `.venv/` to your `.gitignore` so you don’t commit it.

---

## 2) Environment/package managers at a glance (Recommended)

**Why use them?** They help isolate project dependencies, avoid version conflicts, and ensure reproducibility.

Furthermore, in some cases, only a certain set of Python versions will be available on a compute cluster and installing system‑wide Python packages may not be possible (i.e. SCITAS at EPFL). In these cases, environment managers allow you to install and manage packages in user space.

- **pipx** — Installs Python **CLI tools** in isolated environments (e.g., `pipx install pre-commit`). Great for global developer utilities without polluting project envs.
- **conda** — Full environment manager (Python **and** native binaries/CUDA). **Widespread in research**; many scientific binaries are only available via **conda-forge**. Excellent when you need GPU stacks or non-Python deps.
- **poetry** — Dependency manager with lockfiles and `pyproject.toml`; popular in **software development** and **CI/CD** pipelines for reproducibility and clean workflows.
- **uv** — Fast modern toolchain (resolver + virtualenv + runner) with lockfile support; increasingly popular in **software development** and **CI** due to speed and simplicity.

> **Adoption note:** Poetry and uv fit modern software workflows and CI pipelines very well, **but are not yet universally adopted in research**. **Conda** remains dominant in many labs, and **conda-forge** often provides prebuilt packages/binaries that don’t exist on pip.

---

## 3) Conda: install, create envs, install packages, export/recreate YAML

### 3.1 Install Conda
Use a minimal distribution:
- **Miniforge** or **Miniconda**.

**macOS/Linux (bash example):**
```bash
# Download Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O Miniconda3.sh # Linux x86_64
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -O Miniconda3.sh # macOS ARM


# Make installer executable and run it
chmod +x Miniconda3.sh
./Miniconda3.sh

# Initialize shell so `conda` works in new terminals
conda init
# Restart your shell or source your profile
```

**Windows:**
```cmd
# Download Miniconda installer from:
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe --output .\Downloads\Miniconda3.exe
```

- Run the `.exe` installer and follow prompts (install for “Just Me” is fine).
- Open **Anaconda Prompt** or **PowerShell**. If needed, run `conda init`.

Verify:
```bash
conda --version
```

### 3.2 Configure channels
Add **conda-forge** channel for most packages:
```bash
conda config --add channels conda-forge
```

### 3.3 Create & activate an environment
```bash
conda create -n eeg_tutorial python=3.11 -y
conda activate eeg_tutorial
```

List envs:
```bash
conda env list
```

Remove an env:
```bash
conda env remove -n eeg_tutorial
```

### 3.4 Install packages
```bash
pip install numpy pandas matplotlib seaborn scikit-learn ipykernel ipywidgets mne -y
```

### 3.5 Export your environment to YAML
Export **without build pins** and **without the local prefix** so others can reproduce it anywhere.

**macOS/Linux (bash):**
```bash
conda env export --no-builds | sed '/^prefix: /d' > environment.yml
pip freeze >> requirements.txt
```

**Windows (PowerShell):**
```powershell
conda env export --no-builds | Select-String -NotMatch '^prefix:' | Set-Content environment.yml
```

> If you can’t use `sed`/`Select-String`, open the file and manually delete the `prefix:` line.

### 3.6 Create an environment from a YAML file
```bash
conda env create -f environment.yml
# If the env name is in the YAML, conda uses it automatically; otherwise add: -n <name>
```

### 3.7 Update an existing environment from YAML
```bash
conda env update -f environment.yml --prune
```

---

## 4) Best practices checklist

- Example project template: [python-ml-research-template](https://github.com/CLAIRE-Labo/python-ml-research-template)
- One environment **per project**; pin a **Python version** (e.g., 3.11) for stability.  
- For GPU/compiled stacks, prefer **conda** (and **conda-forge**) to get consistent binaries and CUDA builds.  
- Keep project code under `src/` and add `.venv/` (or conda envs) to `.gitignore`.  
- Export reproducible specs: `environment.yml` (conda) or lockfiles (`poetry.lock`, `uv lock`).  
- Avoid committing datasets or secrets; use `.env` for tokens, and keep `data/` out of Git.

---

