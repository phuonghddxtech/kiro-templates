# Project Instructions

## Overview
This project uses Kiro AI as an intelligent agent to assist with development tasks.

## Core Instructions
- Always follow the rules defined in the other steering documents
- Coordinate with sub-agents in `.kiro/agents/` for complex workflows
- Be proactive in identifying potential issues
- Always explain changes before making them
- Prefer incremental changes over large rewrites
- Test assumptions before acting

## Available Agents
Invoke the right agent for each task:

| Agent | File | When to Invoke |
|-------|------|---------------|
| 🖥️ Frontend Developer | `agents/frontend.md` | Components, pages, routing, state, performance |
| 🔧 Backend Developer | `agents/backend.md` | APIs, services, DB queries, background jobs |
| 📋 Project Manager | `agents/project-manager.md` | User stories, sprint planning, status reports |
| 🏗️ Systems Architect | `agents/systems-architect.md` | Architecture decisions, ADRs, system design, scalability |
| 🎨 UI/UX Designer | `agents/ui-ux-designer.md` | Design system, wireframes, UX patterns, accessibility |
| ✅ QA Engineer | `agents/qa.md` | Test plans, writing tests, bug reports, coverage |
| ✍️ Copywriter/SEO | `agents/copywriter-seo.md` | Page copy, microcopy, meta tags, SEO optimization |

## Mandatory Rules (all steering docs apply)
- `clean-code.md` — Clean Code principles
- `code-style.md` — Formatting and naming style guide
- `error-handling.md` — Error classes and global handler patterns
- `tech-stack.md` — Approved technologies
- `system-design.md` — System design patterns
- `project-structure.md` — Folder organization and layered architecture
- `api-conventions.md` — REST API design standards
- `naming-conventions.md` — Cache keys, DB identifiers, queues, events, env vars
- `database.md` — Query patterns, transactions, migrations
- `security.md` — Security requirements (CRITICAL — never violate)
- `monitoring.md` — Logging, metrics, Grafana dashboards, alerting
- `testing.md` — Test coverage standards and patterns
- `git-workflow.md` — Git branching strategy and commit format
