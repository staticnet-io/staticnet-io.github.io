---
title: Setting up a Python Development Environment on macOS
date: 2025-02-02
last_modified_at: 2025-02-02
header:
  overlay_image: /assets/images/500x300.png
  overlay_filter: 0.8
tagline: "A guide to setting up a professional Python development environment" 
excerpt: "Learn how to set up a robust Python development environment on macOS using asdf and virtual environments" 
layout: single
classes: wide
tags: [python, macos, development, tutorial]
---
## Introduction

A properly configured development environment is essential for productive Python development. This guide will help you set up a professional Python workspace using modern tools and best practices. While the core concepts of Python environment management (version control, virtual environments, and dependency isolation) apply across all operating systems, this guide focuses specifically on macOS setup. Users of Linux or Windows may need to adapt these commands and tools for their systems.

It's important to note that macOS and other operating systems come with a system-installed version of Python, but we should typically avoid using it for development. The system Python is often used by the OS itself and various system tools, so modifying it could potentially break system functionality. Additionally, you won't have full control over its version or packages. Instead, you should install and manage a separate Python installation specifically for your development work, which is exactly what we'll cover in this guide.

We'll cover:
- `asdf` for Python version management
- `venv` for Python virtual environments
- A few best practices for Python development on macOS

## Prerequisites

- A relatively current version of macOS
- Basic terminal usage knowledge
- Homebrew package manager

If you haven't installed Homebrew yet, you can do so with:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Installing asdf for Python Version Management

While tools like pyenv are popular, we'll use `asdf` because it provides a consistent interface for managing multiple runtime versions beyond just Python, making it an excellent choice for polyglot development.

### Installation Steps

1. Install asdf:
```bash
brew install asdf
```

2. Add asdf to your shell:
```bash
echo -e '\n. $(brew --prefix asdf)/asdf.sh' >> ~/.zshrc
source ~/.zshrc
```

3. Add the Python plugin:
```bash
asdf plugin-add python
```

4. Install Python:
```bash 
# Install latest stable version:
asdf install python latest
```
```bash 
# OR, specify a version:
asdf install python 3.13.1
```

5. Set your default Python version:
```bash
asdf global python 3.13.1
```

## Using Virtual Environments

Virtual environments are crucial for Python development as they:
- Isolate project dependencies
- Prevent conflicts between different projects
- Make projects more reproducible and portable

### Creating and Managing Virtual Environments

Create a new environment:
```bash
python -m venv myenv
```

Activate the environment:
```bash
source myenv/bin/activate
```

Deactivate when done:
```bash
deactivate
```

### Best Practices

1. Version Control:
   - Add `venv/` to your `.gitignore` file
   - Track dependencies with `requirements.txt`

2. Project Organization:
   - Create one virtual environment per project
   - Name environments meaningfully, especially if storing your venv outside of project folders

## Verifying Your Setup

Verify your installation with these commands:

```bash
# Check Python version
python --version
# This should match the version you set in asdf

# View asdf installed Python versions
asdf list python

# Check virtual environment
which python  # Should show path in venv when activated
pip list      # Show installed packages
```

## Development Environment

While you can use any text editor, Visual Studio Code (VS Code) offers excellent Python support through its Python extension. It provides:
- Integrated debugging
- IntelliSense code completion
- Linting
- Jupyter notebook support

## Conclusion

You now have a robust Python development environment that follows modern best practices. The combination of `asdf` for version management and virtual environments for dependency isolation provides a flexible and maintainable setup. This configuration will serve you well whether you're working on personal projects or professional applications.

Remember to:
- Create new virtual environments for each project
- Keep your dependencies documented
- Update your Python versions as needed using `asdf`

For next steps, consider exploring:
- Setting up linting with tools like `flake8` or `pylint`
- Configuring a test framework like `pytest`
- Adding type checking with `mypy`

Happy coding!

---
*Editor's note: This post was indeed written by me, a human being who goes by the name TJ, but it was revised with assistance from Claude, an AI language model by Anthropic.*