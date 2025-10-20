# Python Virtual Environments & CLI Tools — Beginner-Friendly Notes

These notes explain **virtual environments** (`venv` and `virtualenv`), the meaning of **`python -m`**, and why some packages (like `virtualenv`) give you **terminal commands** after `pip install`.  
At the end you’ll also find a **complete, minimal CLI package** you can build and install so it becomes a command you can run from your shell.

---

## 1) Why virtual environments?

If you install everything globally with **pip**, all projects share the same packages.  
Then upgrading a package for **Project A** can break **Project B**.

A **virtual environment** is a **self-contained folder** with:
- its own **Python interpreter**,
- its own **pip**, and
- its own **site-packages** (installed libraries).

Each project gets its own “bubble,” so versions don’t clash.

---

## 2) Two ways to create virtual environments

You’ll see two approaches in the wild. Both are fine:

### A) Built-in `venv` (Python 3.3+)
- No extra install required.
- Create an environment with the Python you call.

**Create:**  
- Linux/Mac (bash/zsh): **python3 -m venv .venv**  
- Windows (PowerShell/CMD): **py -m venv .venv**

**Activate:**  
- Linux/Mac: **source .venv/bin/activate**  
- Windows PowerShell: **.venv\Scripts\Activate.ps1**  
- Windows CMD: **.venv\Scripts\activate.bat**

**Deactivate:** **deactivate**

### B) `virtualenv` (third-party tool)
- Often a bit faster, supports more scenarios.
- First install it once: **pip install virtualenv**

**Create:**  
- **virtualenv .venv**  
  (or pick a Python: **virtualenv -p /usr/bin/python3.10 .venv**  
  or on Windows: **virtualenv -p C:\Path\To\python.exe .venv**)

**Activate/Deactivate:** same as above.

> ✅ Which should you use?  
> If you already have Python 3, **`python -m venv`** is perfect.  
> If you need special features or speed, **`virtualenv`** is great.

---

## 3) Why do we write `python -m venv`? What does `-m` mean?

**`-m`** tells Python: **“run this standard-library module as a script.”**  
So **`python -m venv .venv`** means: “Use **this exact Python interpreter** to run the **`venv`** module and create an environment in **`.venv`**.”

Why this matters:

- **Correct interpreter**: If you have multiple Pythons installed, **`python -m`** guarantees the module runs with the same interpreter as the `python` you invoked.  
- **No PATH headaches**: You don’t need a separate `venv` executable in your PATH.  
- **Works everywhere**: It’s the most robust, cross-platform way to invoke tools that ship as modules (e.g., **`python -m pip`**, **`python -m http.server`**, **`python -m venv`**).

> Tip: You can (and should) run pip the same way to avoid PATH confusion:  
> **python -m pip install ...**

---

## 4) “I installed `virtualenv` with pip — why is there a new command in my shell?”

Many Python packages include **console scripts**. When you **pip install virtualenv**, pip:
1. installs the package files, and
2. **creates a small executable/wrapper** in your environment’s **Scripts/** (Windows) or **bin/** (Linux/Mac) folder.

That wrapper is what you run as the **`virtualenv`** command.  
This is standard behavior via **“entry points”** (a packaging feature) — you’ll create one yourself in the CLI example below.

So:
- After a **global** install: the script lands in your user/site Scripts/bin, and if that folder is on your **PATH**, the command works everywhere.  
- After a **virtualenv/venv** install: the script is in **`.venv/bin/`** (Linux/Mac) or **`.venv\Scripts\`** (Windows). It appears in your PATH **only while the environment is activated**.

That’s why installing a package can give you a **new shell command** — pip created an executable wrapper for the package’s declared **entry point**.

---

## 5) Recommended folder layout

Two common styles — pick one and be consistent:

**Project-local env (most popular):**
```
my_project/
├─ .venv/               # your environment lives here
├─ requirements.txt
├─ app.py
└─ README.md
```
> Pros: self-contained, easy to onboard others (“activate .venv and go”).

**Central envs folder (alternative):**
```
environments/
├─ project1_env/
└─ project2_env/
my_project/
```
> Pros: all envs in one place. Cons: project and env are separate.

---

## 6) Everyday commands (quick copy list)

- **Create (venv):** **python -m venv .venv**  
- **Activate (Linux/Mac):** **source .venv/bin/activate**  
- **Activate (Windows PS):** **.venv\Scripts\Activate.ps1**  
- **Deactivate:** **deactivate**  
- **Install:** **python -m pip install requests**  
- **Freeze:** **python -m pip freeze > requirements.txt**  
- **Rebuild from file:** **python -m pip install -r requirements.txt**  
- **Delete env (deactivated):** delete the folder **.venv**  
- **Pick Python version (virtualenv):** **virtualenv -p /usr/bin/python3.10 .venv**

---

## 7) Sample workflow (start to finish)

1) **Create the env:** **python -m venv .venv**  
2) **Activate it:** **source .venv/bin/activate** (Linux/Mac) or **.venv\Scripts\Activate.ps1** (Windows)  
3) **Install deps:** **python -m pip install numpy psutil**  
4) **Save exact versions:** **python -m pip freeze > requirements.txt**  
5) **Deactivate when done:** **deactivate**  
6) **On a new machine:** create and activate a new env, then **python -m pip install -r requirements.txt**

---

## 8) Bonus: Build your own CLI tool (so `pip install` gives a command)

Now that you know how **`virtualenv`** becomes a command, let’s make a tiny CLI ourselves.  
We’ll use modern packaging (**PEP 621** with `pyproject.toml`) and define an **entry point** so a command appears after `pip install`.

### 8.1) File structure

```
hello-cli/
├─ pyproject.toml
├─ src/
│  └─ hellocli/
│     ├─ __init__.py
│     └─ cli.py
└─ README.md
```

### 8.2) `pyproject.toml` (declares metadata and the CLI entry point)

```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "hello-cli"
version = "0.1.0"
description = "Tiny example CLI that greets you."
readme = "README.md"
requires-python = ">=3.8"
authors = [{ name = "Your Name", email = "you@example.com" }]
license = { text = "MIT" }
dependencies = []

# This is the magic: expose a shell command named `hello-cli`
# that calls hellocli.cli:main
[project.scripts]
hello-cli = "hellocli.cli:main"
```

### 8.3) `src/hellocli/__init__.py`

```python
__all__ = []
```

### 8.4) `src/hellocli/cli.py` (the actual command code)

```python
import argparse

def main():
    parser = argparse.ArgumentParser(prog="hello-cli", description="Greet someone.")
    parser.add_argument("--name", "-n", default="World", help="Name to greet")
    args = parser.parse_args()
    print(f"Hello, {args.name}!")
```

### 8.5) `README.md` (optional)

```
# hello-cli
A tiny demo CLI. After install, run:

hello-cli --name Alice
```

### 8.6) Build and install locally

From the **hello-cli** folder:

- Make sure you have build tools: **python -m pip install --upgrade build wheel**  
- Build the distribution: **python -m build**  
  (produces `dist/hello-cli-0.1.0.tar.gz` and `dist/hello_cli-0.1.0-py3-none-any.whl`)

- Install your wheel (preferably inside an activated venv):  
  **python -m pip install dist/hello_cli-0.1.0-py3-none-any.whl**

- Now try the command:  
  **hello-cli --name Alice**  
  You should see: **Hello, Alice!**

> Dev tip (editable mode):  
> While developing, install your package in editable mode so changes to code are picked up without rebuilding:  
> **python -m pip install -e .**

> Uninstall:  
> **python -m pip uninstall hello-cli**

**What just happened?**  
When you installed the wheel, `pip` created a **console script** called **`hello-cli`** in your environment’s **bin/** (Linux/Mac) or **Scripts/** (Windows) directory.  
That script forwards to **`hellocli.cli:main`**, so running `hello-cli` calls your `main()` function.  
This is exactly how packages like `virtualenv`, `flake8`, `black`, etc., expose shell commands.

---

## 9) Extra clarity on PATH (why commands sometimes “don’t exist”)

- Global installs write console scripts into a user/site **bin/** or **Scripts/** directory.  
  If that directory is not on your **PATH**, your shell won’t find the command.

- Virtual-env installs write scripts into **.venv/bin/** (Linux/Mac) or **.venv\Scripts\** (Windows).  
  **Activating** the environment temporarily **adds that folder to your PATH**, so the commands appear while it’s active.

If a command works **inside** an activated venv but not globally, it’s usually just a PATH thing, not a broken install.

---

## 10) Final cheat-sheet (safe defaults)

- Prefer **one env per project**, inside the project folder: **.venv**  
- Create with the Python you want: **python -m venv .venv**  
- Always use the interpreter’s pip: **python -m pip ...**  
- Freeze exact versions for reproducibility: **python -m pip freeze > requirements.txt**  
- Rebuild anywhere with: **python -m pip install -r requirements.txt**  
- For CLI tools, define **`[project.scripts]`** in **pyproject.toml** to get a shell command after `pip install`.

You now understand:
- why **`python -m venv`** uses **`-m`**,  
- how a **pip install** can create **terminal commands**, and  
- how to **ship your own** command-line tool the same way.