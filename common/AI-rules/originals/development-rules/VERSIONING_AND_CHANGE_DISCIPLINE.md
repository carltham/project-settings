# Versioning and Change Discipline - MANDATORY

Use semantic versioning for modules and contracts.

## PATCH Version

Clarification or compatible correction with:
- NO new runtime responsibility
- NO schema migration
- NO new endpoint
- NO new event
- NO changed report contract

Example: Bug fixes, documentation updates, typo corrections.

## MINOR Version

Backward-compatible behavior or contract addition:
- Forward schema extension allowed
- New optional parameters
- New optional endpoints
- Existing behavior unchanged

Example: Adding a new feature alongside existing features.

## MAJOR Version

Breaking compatibility:
- Changed module responsibilities
- Removed endpoints
- Changed required parameters
- Substantial new solution surface
- Breaking schema changes

Example: Complete rewrite, removing features, API redesign.

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
