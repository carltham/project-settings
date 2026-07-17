# Document Organization and Structure - MANDATORY

All projects MUST follow strict folder organization for documentation and planning materials.

## Architecture Documents (MANDATORY)

**Location:** `/architecture` folder in project root

**Scope:** System design, architecture decisions, component diagrams, patterns, integration points

**File Naming:** `ARCHITECTURE_*.md` or descriptive names like `DATABASE_SCHEMA.md`, `API_DESIGN.md`

**Examples:**
- `architecture/ARCHITECTURE_PRINCIPLES.md`
- `architecture/API_STANDARDS.md`
- `architecture/DATABASE_SCHEMA.md`
- `architecture/SECURITY_POLICY.md`

## Planning Documents (MANDATORY)

**Location:** `/planning` folder in project root

**Scope:** Sprints, tickets, roadmap, requirements, milestones, feature specifications

**File Naming:** Descriptive names with sprint/ticket reference like `SPRINT_1.md`, `TICKETS.md`

**Examples:**
- `planning/SPRINT_1.md`
- `planning/SPRINT_2.md`
- `planning/TICKETS.md`
- `planning/ROADMAP.md`
- `planning/REQUIREMENTS.md`

## Sub-folders (MANDATORY)

- **Sprints:** `planning/sprints/sprint-1/`, `planning/sprints/sprint-2/`
- **Tickets:** `planning/tickets/` with individual ticket files
- **Architecture Details:** `architecture/layers/`, `architecture/components/` if needed

## README Files (MANDATORY)

- **Root README:** `/README.md` (project overview, quick start, getting started)
- **Architecture README:** `/architecture/README.md` (index of all architecture documents)
- **Planning README:** `/planning/README.md` (index of all planning documents)

## Document Creation Workflow

1. Determine document type: Architecture or Planning
2. Place in correct root folder (`/architecture` or `/planning`)
3. Use descriptive file names
4. Update corresponding README index
5. Link in commit message if document changes existing patterns

## Violations (Never Do These)

- ❌ Architecture documents scattered in root or other folders
- ❌ Planning documents mixed with source code directories
- ❌ Undocumented architectural decisions
- ❌ Sprint information in README files instead of `/planning`
- ❌ Creating new folder structures for documentation without consensus

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
