# Code to Flow Mapping

## Overview

Maps analyzed code modules to generated flows.

## Flow Type Detection Rules

| Indicator | Flow Type |
|-----------|-----------|
| `*.test.*`, `*.spec.*`, `__tests__/` | TDD |
| `components/`, `*.tsx`, `*.vue`, `templates/` | VDD |
| `README.md`, public exports, API docs | DDD |
| Internal logic, no UI, no public API | SDD |

## Mapping Table

| Code Path | Flow | Type | Action | Status | Notes |
|-----------|------|------|--------|--------|-------|
| src/Endpoint.js | flows/legacy/sdd-endpoint/ | SDD | CREATED | DRAFT | Central coordinator, 40+ methods |
| src/Account.js | flows/legacy/sdd-account/ | SDD | CREATED | DRAFT | Immutable data wrapper |
| src/AccountRegistration.js | flows/legacy/sdd-account/ | SDD | CREATED | DRAFT | Registration status |
| src/Call.js | flows/legacy/tdd-call/ | TDD | CREATED | DRAFT | State machine, duration tracking |
| src/Message.js | flows/legacy/sdd-messaging/ | SDD | CREATED | DRAFT | Message data wrapper |
| src/PreviewVideoView.js | flows/legacy/vdd-video/ | VDD | CREATED | DRAFT | Native video component |
| src/RemoteVideoView.js | flows/legacy/vdd-video/ | VDD | CREATED | DRAFT | Native video component |
| index.js | flows/legacy/sdd-endpoint/ | SDD | CREATED | DRAFT | Public API exports |

### Action Values
- **CREATED** - New flow created
- **UPDATED** - Existing flow appended to (additive changes only)
- **UNCHANGED** - Flow exists, no new information found
- **CONFLICT** - Analysis contradicts existing documentation (needs reconciliation)

## ADR Mapping

| Code Pattern | ADR | Type | Status |
|--------------|-----|------|--------|
| EventEmitter usage | PENDING | enabling | DRAFT |
| Promise wrapper pattern | PENDING | enabling | DRAFT |
| Immutable data objects | PENDING | constraining | DRAFT |
| God object (Endpoint) | PENDING | constraining | DRAFT |

## Unmapped (needs manual review)

| Code Path | Reason |
|-----------|--------|
| android/ | Native Android implementation - out of scope |
| ios/ | Native iOS implementation - out of scope |
| examples/ | Usage examples - documentation, not implementation |

---

*Auto-generated. Update as analysis progresses.*
