# Development Standards

## Coding Principles

SOLID

KISS

DRY

YAGNI

---

# Architecture Rules

UI cannot access database directly.

Services cannot depend on UI.

Domain must remain independent.

---

# Naming Convention

PascalCase

Example:

GameService

SearchService

LaunchHistory

---

# Folder Rules

One Responsibility Per Folder

One Responsibility Per Class

---

# Logging

Use Structured Logging

Levels:

Debug

Info

Warning

Error

Critical

---

# Git Workflow

main

develop

feature/*

hotfix/*

---

# Commit Format

feat:

fix:

refactor:

docs:

test:

chore:

---

# Testing

Unit Tests Required

Integration Tests Recommended

Manual UI Tests Required

---

# Performance Rules

No blocking UI thread

No synchronous image loading

No heavy startup initialization
