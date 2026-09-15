# Cem Çakırlar - Universal Agent Skills (`cc-*`)

This repository contains custom, universal AI agent skills compliant with the **`npx skills` standard (Single Source of Truth)**. Every skill is prefixed with **`cc-`** for instant namespace recognition and zero collisions with standard/community packages.

Compatible with **Claude Code**, **Google Antigravity**, **Cursor**, **Codex / OpenAI**, **VS Code**, and any AI assistant supporting universal agent skills.

---

## 📦 Included Skills

| Skill | Description | Category |
| :--- | :--- | :--- |
| **`cc-agent-env-doctor`** | Universal skills manager & environment auditor. Handles safe soft-delete archiving, restoring, updates, and ecosystem inspection. | Tooling / Env Management |
| **`cc-create-tests`** | Create, structure, and write high-resilience tests under `tests/unit\|integration\|e2e` adhering to strict anti-colocation and type safety. | Testing & Quality |
| **`cc-doc-audit`** | Audit project and agent documentation for contradictions, code drift, and dead paths. | Documentation & Hygiene |

---

## 🚀 Installation

### 1. Install All Skills Globally (Recommended)
To install all `cc-*` skills cleanly into your universal agent skills pool (`~/.agents/skills`):

```bash
npx -y skills add cemcakirlar/cemcakirlar-skills -g --skill '*' -a universal -y
```

### 2. Install a Specific Skill
To install a specific skill (e.g. `cc-agent-env-doctor`):

```bash
npx -y skills add cemcakirlar/cemcakirlar-skills --skill cc-agent-env-doctor -g -a universal -y
```

---

## 🔄 Updating Skills

When updates are published to this repository, update them across all your agents with:

```bash
npx skills update
```
*(Or run `python3 ~/.agents/skills/cc-agent-env-doctor/scripts/audit_environment.py --update-skills`)*

---

## 🛠 Directory Structure

```text
cemcakirlar-skills/
├── README.md
├── .gitignore
└── skills/
    ├── cc-agent-env-doctor/
    │   ├── SKILL.md
    │   └── scripts/
    │       └── audit_environment.py
    ├── cc-create-tests/
    │   └── SKILL.md
    └── cc-doc-audit/
        └── SKILL.md
```

---

## 📄 License
MIT
