---
layout: post
title: "Part 3 - GitHub Actions CI Pipeline: Automating Quality Assurance"
date: 2025-10-11 19:00 -0300
permalink: /posts/github-actions-ci-cd-pipeline/
categories: [CI, build]
tags: [CI, GitHub Actions, DevOps, build]
---

<link rel="stylesheet" href="/assets/css/custom.css">

## 🚀 Welcome Back to Our Production-Ready Journey

Hello again! In [Part 1](/posts/welcome/), we explored hexagonal architecture, and in [Part 2](/posts/unit-testing-hexagonal-architecture/), we added unit testing. Today, we're taking a crucial step toward production readiness: **automating our quality assurance** with GitHub Actions CI pipeline.

If you've ever wondered how professional teams ensure code quality at scale, this is it. We're moving from "it works on my machine" to "it works everywhere, automatically."

# Car Object Detection Case Study Part 3: GitHub Actions CI Pipeline

## 🎯 Why CI Matters (And Why Manual Testing Isn't Enough)

Continuous Integration (CI) is the backbone of modern software development. It provides **automated quality gates** that catch issues before they reach production, ensure consistent environments across development and production, and enable **confident code integration** at any time.

### The Manual Testing Trap

Here's what happens in most projects without CI:

```bash
# The "manual testing" approach
$ python -m pytest tests/
# ✅ Tests pass locally

$ git push origin main
# 🚀 Code is now in production

# Two days later...
$ python -m pytest tests/
# ❌ Tests fail! What changed?
```

**The problems with manual testing:**
- **Environment differences** (Python version, dependencies, OS)
- **Forgotten test runs** (developers skip testing when in a hurry)
- **Inconsistent results** (works on Mac, fails on Linux)
- **Late bug discovery** (issues found in production, not development)

### The CI Solution

With GitHub Actions, every pull request triggers an automated pipeline that:
- **Runs tests** in a clean, consistent environment
- **Provides immediate feedback** to developers
- **Prevents broken code** from reaching the main branch

## 🏗️ Our CI Implementation: Focused Testing Pipeline

Let's explore how we've implemented a focused CI pipeline for our car detection project:

### 1. Workflow Structure: The Foundation

Our GitHub Actions workflow follows a **simple and focused approach**:

```yaml
# .github/workflows/continuous_integration.yml
name: Continuous Integration

on:
  pull_request:
    branches:
      - main

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
      
      - name: Run tests with coverage
        run: uv run pytest --cov=src --cov-report=term --cov-report=html
```

**Key Benefits:**
- ✅ **Consistent environment** (Ubuntu latest)
- ✅ **Fast dependency management** with uv
- ✅ **Coverage reporting** (terminal + HTML)
- ✅ **Pull request integration**

### Why uv for Dependency Management?

We chose **uv** over traditional pip for several reasons:
- **Speed**: uv is significantly faster than pip for dependency resolution
- **Built-in caching**: Automatic caching reduces CI execution time
- **Modern Python tooling**: Designed for modern Python projects
- **Simplified workflow**: Less configuration needed compared to pip + virtualenv



## 📊 Workflow Results: Real-Time Quality Feedback

Let's look at what our CI pipeline produces:

![Workflow Completed](/assets/img/workflow_completed.png)

**What this tells us:**
- ✅ **All tests passed** 
- ✅ **Ready for integration**

### Pull Request Integration

When we create a pull request, the CI pipeline runs automatically:

![Workflow Approved](/assets/img/workflow_approved.jpg)

**Benefits:**
- **Immediate feedback** on code changes
- **Quality gates** before merging
- **Confidence in code** before review
- **Documentation** of what was tested

## 📈 Monitoring and Metrics

### 1. **Test Coverage Tracking**

![Approved Unit Test in CI](/assets/img/approved_unittest_in_ci.jpg)

**Coverage Metrics:**
- **Branch coverage**: 89% (good)
- **Trend tracking**: Coverage maintained over time

## 🎓 Key Takeaways

1. **CI is essential**—not optional for production software
2. **Quality gates** prevent bad code from reaching production
3. **Automated testing** ensures consistent environments
4. **Automation** reduces human error and increases confidence
5. **Fast feedback** improves developer productivity

## 🚀 What's Coming Next?

In the next post, we'll explore:

### **Phase 1: Foundation** 🏗️
- **Part 4**: Formatter and linter

### **Phase 2: Coming Soon** 🚀
*Stay tuned for exciting new features and production-ready enhancements!*

We'll add code formatting and linting tools to ensure consistent code style across the project.


## 📚 Resources and Next Steps

- **Repository up to this point**: [object_detection_repo](https://github.com/Guiandreis/car_detection_repo/tree/41ebd608bab5d84591572304120e772d5ecebdad)
- **Pull request with CI changes**: [PR #9](https://github.com/Guiandreis/car_detection_repo/pull/9)
- **GitHub Actions documentation**: [docs.github.com/actions](https://docs.github.com/en/actions)

---


*This is Part 3 of a multi-part series on building production-ready Python applications. Stay tuned for more!*
