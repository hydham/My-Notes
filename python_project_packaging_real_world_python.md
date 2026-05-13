# Python Project Packaging with pyproject.toml

So one of those topics that I've always had some niggling confusion about — how to set up your Python project when that little script you've been working on turns into something a bit larger.

As it turns out, this has implications for how you call your Python modules from the command line, how you structure your imports, how easy it is to configure tools like a code formatter, and when the time comes — how to package it for distribution.

One of the pain points I've had when working with Python is that sometimes you end up having to do a lot of navigating on the command line to call Python from the right place — or else your imports won't work. Also, being able to call a Python utility project from anywhere with a simple command can end up being a bit of a rabbit hole.

Most of the time these tasks end up being more complicated than you might expect because there's seemingly many ways to go about it.

What we're going to cover today is the Python-sanctioned way to achieve all these things easily — by setting up a `pyproject.toml` file and creating an editable install of your package.

Using a `pyproject.toml` file and creating an editable install may sound like something you might only need when distributing your package. However, it's something that will make developing anything larger than a single file script much easier and less headache-prone.

---

## Where We're Starting

Our starting point is a little CLI app that will basically echo whatever you pass into it.

```bash
python cli.py sleep tight little man club
```

And then a snake says back to you:

```
sleep tight little man club
```

The entry point file `cli.py` takes the arguments that were passed in and calls a `say` function from the `snake` module.

Here's `snake.py`:

```python
# ASCII art snake drawing
# bubble() — wraps the message in a speech bubble
# say() — prints the bubble and the snake
```

And `cli.py` just calls `snake.say()` with whatever arguments were passed in.

Currently the structure is just these two files sitting flat — no folders, no package. That's what we're going to fix.

---

## Step 1 — Give the Project a Real Name

Right now we're calling a program called `cli` — which is a weird abbreviation for command line interface. It's not really saying anything about the project.

We could rename `cli.py` to something better. But it actually is describing the role that module plays. An even better solution is to create a directory that contains our project.

```bash
mkdir snake_say
```

Move both files into it:

```
snake_say/
    cli.py
    snake.py
```

Now from the command line:

```bash
python snake_say
```

That's not going to work — yet.

```bash
python snake_say/cli.py sleep tight go mom
```

That still works, but it's not ideal. We've got more to type and we're still calling `cli` explicitly.

---

## Step 2 — `__main__.py`

Python has a way for you to call a directory instead of a file. To do this, Python needs to know where the entry point is inside that directory. It looks for a file called `__main__.py`.

So let's rename `cli.py` to `__main__.py`.

```
snake_say/
    __main__.py
    snake.py
```

Now:

```bash
python snake_say sleep tight go mom
```

That works. We no longer have to reference the `cli` file at all. Python sees the directory, finds `__main__.py`, and runs it.

`__main__.py` files might look a little weird and scary at first — but they are a convention you'll use everywhere and get used to. This is where the entry point of your program lives. This is where your program starts running.

---

## Step 3 — The `-m` Flag Problem

You'll often see people use the `-m` flag:

```bash
python -m snake_say sleep tight go mom
```

But if we try this we get a `ModuleNotFoundError`:

```
ModuleNotFoundError: No module named 'snake'
```

It's failing on the `import snake` line in `__main__.py`. Why?

The difference is how Python handles the path.

When you run `python snake_say` — Python adds the `snake_say` directory to its path. So it can find `snake.py` sitting right there inside it.

When you run `python -m snake_say` — Python looks in its installed packages path, not the local file system the same way. The `snake_say` directory itself doesn't get added to path. So `import snake` fails because `snake.py` isn't findable.

---

## Step 4 — Absolute Imports (The First Fix)

The fix for `-m` is to use an **absolute import** — explicitly saying where `snake` lives:

```python
# __main__.py
from snake_say import snake
```

Now test it:

```bash
python -m snake_say sleep tight go mom
```

That works. But now let's try the original way:

```bash
python snake_say sleep tight go mom
```

Broken. We've flip-flopped the problem.

This is the core tension here — each way of calling the script adds a different directory to Python's path. One finds `snake`, the other finds `snake_say`. You can't satisfy both with a plain import.

---

## The Wrong Solution — Messing with `sys.path`

You might be tempted at this point to just manually add to Python's path:

```python
import sys
sys.path.append("C:\\Users\\YourName\\real_python\\snake_say")
```

This works. You'll see it recommended in a lot of places. But it doesn't scale:

- If you have multiple projects talking to each other, you keep adding to path
- If someone else has a different file layout, they have to go in and change all the paths
- Different operating systems, different drives — it gets messy fast

Stay completely away from this. We're going to play on the same team as Python's import system, not sneak around it.

---

## The Right Solution — Install the Package

What we want to do is `pip install` our own package. Locally. No uploading to PyPI required.

Most of the time you use `pip install` for things like pandas or Django — it goes out to the internet. But you can also just say:

```bash
pip install ./snake_say
```

And it installs from your local directory. No internet. No one sees your code.

After you do this, `snake_say` is available from anywhere you run Python in that environment — just like Django or pandas.

But there's a catch: when you're developing, you don't want to reinstall every time you change a line of code. That's where the **editable install** comes in.

---

## Step 5 — `pyproject.toml`

Before we can install anything, we need to tell Python about our project. That lives in a file called `pyproject.toml`.

First, let's reorganize. Create a top-level project folder:

```
snake_say_project/
    snake_say/
        __main__.py
        snake.py
    pyproject.toml
```

The `pyproject.toml` sits **outside** the `snake_say` package folder, at the project root. This is where your code lives, and `pyproject.toml` is the definition of it.

---

## A Brief History (So You Know Why We're Here)

This file replaces the old `setup.py` and `setup.cfg` files. If you've done packaging before, you've seen those. We don't need them anymore — just this one file.

Python packaging has a messy history. Back when Python was first created — roughly the same time as the World Wide Web — there was no thinking about code distribution at all. People shared single files by email.

Then came `distutils` (Python 2 era) — now deprecated and being removed in Python 3.12. Then came `setuptools` as a replacement. For many years that was the only way to package things.

Eventually the community started defining PEPs — standards for how packaging should work. The place we've arrived at now is actually very neat and clean. `pyproject.toml` is the standard. It's defined in PEP 621.

---

## Writing `pyproject.toml`

Start with the build system section — you can copy this directly:

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.backends.legacy:build"
```

This tells pip which build system to use. `setuptools` is the most common and has the most history. Others like `flit` and `poetry` are also options — each with different strengths — but `setuptools` is what you want here.

Now add the project metadata:

```toml
[project]
name = "snake_say"
version = "1.0.0"
```

That's the minimum required. `name` and `version`. Everything else is optional at this stage — those extra fields matter more when you're distributing publicly.

Save the file.

---

## Step 6 — Create a Virtual Environment

Before installing, set up a virtual environment:

```bash
python -m venv .venv
```

Activate it:

```bash
# Windows
.venv\Scripts\activate

# Mac/Linux
source .venv/bin/activate
```

---

## Step 7 — Editable Install

From inside `snake_say_project/`:

```bash
pip install -e .
```

The `.` means "install this directory." The `-e` flag means **editable** — instead of freezing a copy of your code, pip points directly at your source files. Any change you make shows up immediately. No reinstalling.

You'll see output like:

```
Successfully installed snake-say-1.0.0
```

Now test both ways:

```bash
python snake_say sleep tight go mom
```

```bash
python -m snake_say sleep tight go mom
```

Both work. And from anywhere:

```bash
cd ..
python -m snake_say hello from outside
```

Still works. That's the power of installing a package — Python's path now knows about it everywhere in this environment.

---

## Step 8 — A Real Terminal Command

Let's go one step further. Instead of typing `python -m snake_say`, what if you could just type:

```bash
snakey hello there
```

That's possible with `[project.scripts]` in `pyproject.toml`:

```toml
[project.scripts]
snakey = "snake_say.__main__:main"
```

The format is `command_name = "package.module:function"`.

But wait — we don't have a `main` function in `__main__.py` yet. Let's add one:

```python
# __main__.py
import sys
from snake_say import snake

def main():
    args = sys.argv[1:]
    snake.say(" ".join(args))

if __name__ == "__main__":
    main()
```

That `if __name__ == "__main__"` block at the bottom is important — we'll explain why in a moment.

Now reinstall — because we changed `pyproject.toml`, the editable install won't pick that up automatically:

```bash
pip install -e .
```

Now:

```bash
snakey sleep tight go mom
```

One snake. From anywhere. Beautiful.

---

## The `if __name__ == "__main__"` Idiom

Without that check, you'd see two snakes. Here's why.

When `snakey` runs, Python:
1. Imports `__main__.py` — which runs the file top to bottom
2. Calls the `main` function as specified in `pyproject.toml`

Without the guard, step 1 already calls `main()` once. Then step 2 calls it again. Two snakes.

The `__name__` variable is set to `"__main__"` only when a file is run directly, not when it's imported. So:

```python
if __name__ == "__main__":
    main()
```

This says — only call `main()` if I'm being run directly, not if I'm being imported. One snake. Always.

---

## What the `.egg-info` Folder Is

You'll notice a `snake_say.egg-info` folder appeared. Don't worry about it — it's metadata from the install. It contains things like your console script pointers and version info.

You can inspect it from Python:

```python
from importlib.metadata import metadata
print(metadata("snake_say")["Version"])
# 1.0.0
```

It reads right from that egg-info. Add it to your `.gitignore` — you don't want to commit it:

```
*.egg-info/
```

But do commit `pyproject.toml`. That's where all the important information lives.

---

## Final Project Structure

```
snake_say_project/
    .venv/
    snake_say/
        __init__.py
        __main__.py
        snake.py
    pyproject.toml
```

And `pyproject.toml` in full:

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "snake_say"
version = "1.0.0"

[project.scripts]
snakey = "snake_say.__main__:main"
```

---

## The Takeaway

We took two flat files and turned them into a properly packaged Python project. Here's what that gives you:

- Consistent imports everywhere — always `from snake_say import snake`, regardless of where you're calling from
- No messing with `sys.path` ever
- A real terminal command (`snakey`) available anywhere in your environment
- A foundation ready for distribution if you ever want to publish it

This is the way. It's defined by PEPs, supported by pip, and works today. Old `setup.py` and `setup.cfg` still work — but they're on their way out. `pyproject.toml` is where Python is going.

Start using it even for personal projects. You'll thank yourself later.
