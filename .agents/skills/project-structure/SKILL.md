---
name: project-structure
description: Standard folder layout, layered architecture, and file naming conventions for all projects. Invoke when creating new projects, organizing code, or reviewing folder structure.
---

# Project Structure Skill

> Standard folder organization and layered architecture for all projects.

## Standard Folder Layout

```
project-root/
├── .agents/                    # Antigravity Agent configuration
│   ├── skills/                 # Specialized AI skills
│   └── workflows/              # Reusable command workflows
│
├── .claude/                    # Claude Code configuration
│   ├── agents/                 # Sub-agent definitions
│   ├── commands/               # Reusable command workflows
│   ├── rules/                  # Mandatory rules for AI
│   ├── skills/                 # Specialized AI skills
│   └── CLAUDE.md               # Main AI instructions
│
├── src/                        # Application source code
│   ├── config/                 # Configuration files
│   ├── controllers/            # Route handlers (thin layer)
│   ├── middleware/             # Express middleware
│   ├── models/                 # Database models/schemas
│   ├── repositories/           # Data access layer
│   ├── routes/                 # Route definitions
│   ├── services/               # Business logic layer
│   ├── utils/                  # Utility functions
│   └── index.js                # Application entry point
│
├── tests/                      # Test files
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                    # End-to-end tests
│
├── docs/                       # Documentation
│   ├── api/                    # API documentation
│   └── architecture/           # Architecture diagrams (Mermaid)
│
├── scripts/                    # Build and utility scripts
├── .env.example                # Example environment variables
├── .gitignore                  # Git ignore rules
├── package.json
└── README.md
```

---

## Layered Architecture
```
Request → Routes → Middleware → Controllers → Services → Repositories → Database
```

| Layer | Responsibility |
|-------|---------------|
| **Routes** | URL mapping only, no logic |
| **Middleware** | Auth, validation, rate-limit, logging |
| **Controllers** | Request/response handling, input validation — **thin layer** |
| **Services** | Business logic, orchestration |
| **Repositories** | Data access, queries |
| **Models** | Data schemas and types |

---

## File Naming
- Source files: `kebab-case.js` (`user-service.js`, `auth-middleware.js`)
- Test files: `[name].test.js` (`user-service.test.js`)
- Config files: `kebab-case.js` or `kebab-case.json`
- Component files (React/RN): `PascalCase.tsx` (`OrderCard.tsx`, `Button.tsx`)

---

## Environment Files
- `.env` — Local development (gitignored)
- `.env.example` — Template committed to git
- `.env.test` — Test environment (gitignored)
- `.env.production` — Set in CI/CD, never committed

---

## Feature-Based Organization (Alternative)
For larger projects, organize by feature instead of layer:

```
src/
├── features/
│   ├── users/
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   ├── user.validator.ts
│   │   ├── user.types.ts
│   │   └── user.test.ts
│   ├── orders/
│   │   ├── order.controller.ts
│   │   ├── order.service.ts
│   │   └── ...
│   └── payments/
├── shared/                     # Cross-cutting concerns
│   ├── middleware/
│   ├── utils/
│   └── types/
└── main.ts
```

> **Rule**: Use layered for small projects, feature-based for larger ones. Never mix both in the same project.
