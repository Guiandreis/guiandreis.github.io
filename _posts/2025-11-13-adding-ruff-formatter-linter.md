---
layout: post
title: "Part 4 - Adding Ruff as Formatter and Linter: Enforcing Code Quality"
date: 2025-11-13 19:00 -0300
categories: [Code Quality, build]
tags: [Ruff, Linting, Formatting, Code Quality, build]
---

<link rel="stylesheet" href="/assets/css/custom.css">

## 🚀 Welcome Back to Our Production-Ready Journey

Hello again! In [Part 1](/posts/welcome/), we explored hexagonal architecture, in [Part 2](/posts/unit-testing-hexagonal-architecture/), we added unit testing, and in [Part 3](/posts/github-actions-ci-cd-pipeline/), we automated our CI pipeline. Today, we're taking another crucial step toward production readiness: **enforcing consistent code quality** with Ruff as our formatter and linter.

If you've ever worked on a team where code style was inconsistent, you know the pain. But with Ruff, we're ensuring that every line of code follows best practices—automatically.

# Car Object Detection Case Study Part 4: Adding Ruff as Formatter and Linter

## 🎯 Why Code Formatting and Linting Matter (And Why Most Projects Skip It)

Code formatting and linting are often seen as "nice-to-have" features, but they're actually **essential for maintainability**. They provide **consistent code style** across the entire codebase, catch potential bugs before they reach production, and make code reviews focus on logic rather than style. More importantly, automated formatting and linting enable **faster development**—you don't waste time debating whether to use single or double quotes, or where to put that closing brace.

### The Inconsistent Code Problem

Here's what happens in most projects without automated formatting and linting:

```python
# Developer A writes this:
def detect_cars(image_path:str)->list:
    model = YOLO('yolov8n.pt')
    results = model(image_path)
    cars = [box for box in results[0].boxes if box.cls == 2]
    return cars

# Developer B writes this:
def detect_cars(image_path: str) -> list:
    model = YOLO("yolov8n.pt")
    results = model(image_path)
    cars = [box for box in results[0].boxes if box.cls == 2]
    return cars

# Developer C writes this:
def detect_cars( image_path: str ) -> list:
    model = YOLO( 'yolov8n.pt' )
    results = model( image_path )
    cars = [ box for box in results[ 0 ].boxes if box.cls == 2 ]
    return cars
```

**The problems with inconsistent code:**
- **Style debates** waste time in code reviews
- **Hard to read** code slows down development
- **Hidden bugs** from unused imports and variables
- **Technical debt** accumulates over time

### The Ruff Solution

With Ruff, we get:
- **Automatic formatting** (no more style debates)
- **Fast linting** (catches bugs before they reach production)
- **Single tool** (replaces Black, Flake8, isort, and more)
- **Pre-commit hooks** (enforces quality before commits)

## 🏗️ Our Ruff Implementation: Unified Code Quality

Let's explore how we've integrated Ruff into our car detection project:

### 1. Why Ruff? The Modern Python Tool

Ruff is a **blazingly fast** Python linter and formatter written in Rust. It's designed to replace multiple tools (Black, Flake8, isort, pyupgrade, and more) with a single, fast tool.

**Why we chose Ruff:**
- **Speed**: 10-100x faster than traditional tools
- **All-in-one**: Replaces multiple tools with one
- **Modern**: Built for Python 3.10+ with modern features
- **Configurable**: Easy to customize for our needs

### 2. Configuration: Setting Up Ruff

We configured Ruff in our `pyproject.toml` file with a comprehensive set of rules. Here are the key configuration decisions:

```toml
[tool.ruff]
target-version = "py311"
line-length = 100
indent-width = 4

[tool.ruff.lint]
# Enable 50+ rule categories covering:
# - Code style (E, W, F, I, N)
# - Security (S, BLE)
# - Performance (PERF)
# - Type checking (TCH, FA)
# - Testing (PT)
# - And many more...
select = ["E", "W", "F", "UP", "B", "SIM", "I", "N", "D", "S", ...]

# Ignore overly strict rules
ignore = [
    "D100", "D104",  # Docstring requirements
    "S101",          # Assert usage
    "PLR0913",       # Too many arguments
    "COM812",        # Conflicts with formatter
]

# Allow unused variables when underscore-prefixed
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["S101", "ARG", "FBT", "D"]  # Relaxed rules for tests
"main.py" = ["T201"]  # Allow print() in main script

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.ruff.lint.isort]
known-first-party = ["adapters", "config", "domain", "ports"]

[tool.ruff.format]
quote-style = "double"
exclude = ["tests/**/*.py"]  # Don't format test files
```

**Key Configuration Decisions:**
- **Line length**: 100 characters (more generous than Black's 88)
- **Comprehensive rules**: 50+ rule categories enabled for maximum code quality
- **Smart ignores**: Relaxed rules for tests and main script
- **Google docstrings**: Enforced via pydocstyle
- **Import organization**: Configured isort with our hexagonal architecture modules
- **Test exclusion**: Tests excluded from formatting to preserve test patterns

### 3. Pre-commit Integration: Quality at Commit Time

We integrated Ruff with pre-commit hooks to ensure code quality before commits:

```yaml
# .pre-commit-config.yaml
default_install_hook_types: [pre-commit, pre-push]

repos:
  # Basic quality checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: check-yaml
      - id: check-toml
      - id: end-of-file-fixer
      - id: check-added-large-files
        args: [--maxkb=10000]  # Prevent files >10MB
      - id: check-merge-conflict

  # Ruff formatting and linting (official hooks)
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.4  # Check for latest version
    hooks:
      - id: ruff-format  # Auto-format code
        stages: [pre-commit]

      - id: ruff         # Lint with auto-fix
        args: [--fix, --exit-non-zero-on-fix]
        stages: [pre-commit]

  # Run tests before push
  - repo: local
    hooks:
      - id: tests
        name: Run unit tests
        entry: uv run pytest --cov=src --cov-report=term --cov-report=html
        language: system
        stages: [pre-push]
        verbose: true
        require_serial: true
        pass_filenames: false
```

**Pre-commit Benefits:**
- **Multi-stage hooks**: Pre-commit for formatting/linting, pre-push for tests
- **Automatic fixes**: Ruff formats and fixes issues automatically before commit
- **File validation**: Checks YAML/TOML syntax, prevents large files (>10MB), detects merge conflicts
- **Code quality**: Ensures consistent formatting and catches linting issues before commit
- **Test safety**: Runs full test suite before push to prevent broken code from being pushed
- **Developer-friendly**: Fast feedback at commit time, comprehensive checks before push

### 4. Running Ruff: Developer Workflow

Developers can run Ruff manually when needed:

```bash
# Check and auto-fix issues
ruff check --fix .

# Format code
ruff format .
```

**Workflow Integration:**
- **Before committing**: Pre-commit hooks run automatically
- **In CI**: Automated checks on every pull request

## 📊 Impact: Before and After

Let's look at the impact of adding Ruff to our project:

### Code Changes Summary

From [PR #10](https://github.com/Guiandreis/car_detection_repo/pull/10):
- **+375 lines added** (Ruff configuration, pre-commit setup)
- **-211 lines removed** (Cleaned up code, removed unused imports)
- **Net improvement**: Cleaner, more consistent codebase

### What Changed in Our Code

**1. Import Organization:**
```python
# Before (inconsistent)
from ultralytics import YOLO
import cv2
from typing import List, Tuple
import os

# After (organized by Ruff)
import os
from typing import List, Tuple

import cv2
from ultralytics import YOLO
```

**2. Formatting Consistency:**
```python
# Before (inconsistent spacing)
def filter_cars(detections:List[tuple])->List[tuple]:
    return [d for d in detections if d[-1]==2]

# After (consistent formatting)
def filter_cars(detections: List[tuple]) -> List[tuple]:
    return [d for d in detections if d[-1] == 2]
```

**3. Removed Unused Code:**
- **Unused imports** removed automatically
- **Unused variables** flagged and removed
- **Dead code** eliminated

### Test Code Improvements

Ruff also improved our test code:

```python
# Before
def test_filter_cars():
    detections = [(0.9, 0.1, 0.2, 0.95, 1), (0.8, 0.3, 0.4, 0.87, 2)]
    result = filter_cars(detections)
    assert result==[(0.8, 0.3, 0.4, 0.87, 2)]

# After (consistent formatting)
def test_filter_cars():
    detections = [(0.9, 0.1, 0.2, 0.95, 1), (0.8, 0.3, 0.4, 0.87, 2)]
    result = filter_cars(detections)
    assert result == [(0.8, 0.3, 0.4, 0.87, 2)]
```

## 🎯 Benefits We've Gained

### 1. **Consistency Across the Codebase**

Every file now follows the same style:
- **Consistent spacing** around operators
- **Uniform quote style** (double quotes)
- **Organized imports** (standard library, third-party, known-first-party, local)
- **Consistent line length** (100 characters)
- **Google-style docstrings** enforced across the codebase

### 2. **Faster Code Reviews**

Code reviews now focus on logic, not style:
- **No style debates** (Ruff enforces it)
- **Faster reviews** (less time on formatting)
- **Better feedback** (focus on architecture and logic)


### 3. **Developer Experience**

Developers benefit from:
- **Automatic fixes** (Ruff fixes most issues automatically)
- **Fast feedback** (Ruff is extremely fast, even with 50+ rule categories)
- **Less cognitive load** (no style decisions needed)
- **Better IDE integration** (works with VS Code, PyCharm, and other editors)
- **Flexible rules**: Per-file ignores allow exceptions where they make sense

## 🚀 Integration with CI Pipeline

Ruff integrates seamlessly with our existing CI pipeline:

```yaml
# .github/workflows/continuous_integration.yml
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
      
      - name: Install dependencies
        run: uv sync
      
      - name: Run Ruff check
        run: uv run ruff check .
      
      - name: Run Ruff format check
        run: uv run ruff format --check .
      
      - name: Run tests with coverage
        run: uv run pytest --cov=src --cov-report=term --cov-report=html
```

**CI Integration Benefits:**
- **Quality gates**: Code must pass Ruff checks before merging
- **Consistent enforcement**: Same rules in CI and locally
- **Fast checks**: Ruff runs quickly even with 50+ rule categories

## 🎓 Key Takeaways

1. **Code quality tools are essential**—not optional for production software
2. **Ruff replaces multiple tools**—faster and simpler than Black + Flake8 + isort + 20+ other tools
3. **Comprehensive rules catch more issues**—50+ rule categories catch bugs, security issues, and performance problems
4. **Pre-commit hooks enforce quality**—catch issues before they reach the repository
5. **Smart configuration is key**—per-file ignores and sensible exceptions make rules practical
6. **Consistency improves readability**—easier to understand and maintain with uniform style
7. **Automated formatting saves time**—no more style debates in reviews
8. **Fast feedback improves productivity**—Ruff is extremely fast even with comprehensive rules

## 🚀 What's Coming Next?

In the next post, we'll explore:

### **Phase 1: Foundation** 🏗️
- **Part 5**: Introduction of a Makefile

We'll add a Makefile to standardize common development tasks and make our workflow more efficient.

### **Phase 2: Coming Soon** 🚀
*Stay tuned for exciting new features and production-ready enhancements!*


## 📚 Resources and Next Steps

- **Repository up to this point**: [car_detection_repo](https://github.com/Guiandreis/car_detection_repo/tree/aa73d8f)
- **Pull request with Ruff changes**: [PR #10](https://github.com/Guiandreis/car_detection_repo/pull/10)
- **Ruff documentation**: [docs.astral.sh/ruff](https://docs.astral.sh/ruff/)
- **Pre-commit documentation**: [pre-commit.com](https://pre-commit.com/)
- **Python Code Quality**: [realpython.com/python-code-quality](https://realpython.com/python-code-quality/)

---

**Ready to dive deeper?** In the next post, we'll continue building our production-ready foundation with more quality tools and practices.

*This is Part 4 of a multi-part series on building production-ready Python applications. Stay tuned for more!*
