# ADR-004: God Object Pattern

> Architectural Decision Record for centralized Endpoint class vs modular design.

**Status**: DRAFT  
**Date**: 2026-03-04  
**Level**: Constraining  
**Source**: Legacy analysis of src/Endpoint.js (580+ lines)

---

## Context

The SIP library needs to coordinate multiple responsibilities:
- Native module initialization
- Account management (create, register, delete)
- Call operations (make, answer, hold, transfer, DTMF, etc.)
- Messaging (send, receive, typing indicators)
- Event routing (7 native events)
- Configuration (network, codec, orientation)

Design options:
1. **God Object**: Single class handles everything
2. **Modular Design**: Separate classes for each responsibility
3. **Facade Pattern**: Simple interface over complex subsystem
4. **Micro-services**: Event-bus based decoupled modules

---

## Decision

**Use God Object pattern: Endpoint handles all responsibilities.**

Endpoint.js (580+ lines) contains:
- 40+ public methods
- 7 private event handlers
- 1 utility method (_normalize)
- Constructor with 7 event subscriptions

```javascript
export default class Endpoint extends EventEmitter {
  constructor() { /* 7 event listeners */ }
  start() { /* initialization */ }
  createAccount() { /* account management */ }
  makeCall() { /* call operations */ }
  sendMessage() { /* messaging */ }
  // ... 40+ methods total
}
```

---

## Rationale

**Why God Object (in this case):**

1. **Native Bridge Simplicity**: Single point of contact with native module
2. **Centralized Event Handling**: All events flow through one place
3. **Easy to Use**: Application only needs one object (endpoint)
4. **Legacy Code**: Decision already made, analyzing existing implementation
5. **PjSIP Architecture**: Native library is also monolithic

**Why NOT alternatives:**

- **Modular Design**: More files, more complexity, overkill for this scope
- **Facade Pattern**: Would still need underlying modules, adds indirection
- **Micro-services**: Over-engineering for mobile library

---

## Consequences

### Positive (+)

- ✅ Single entry point (easy to understand)
- ✅ Centralized event routing
- ✅ Consistent API (all methods in one place)
- ✅ Easy dependency injection (Endpoint is singleton candidate)
- ✅ Direct mapping to native module

### Negative (-)

- ❌ High complexity (580+ lines in one file)
- ❌ Single point of failure
- ❌ Hard to test (many responsibilities)
- ❌ Violates Single Responsibility Principle
- ❌ Difficult to extend (file keeps growing)
- ❌ Tight coupling (all methods depend on Endpoint instance)

### Neutral (○)

- ○ All methods must be on Endpoint instance
- ○ Event handlers are private (naming convention)
- ○ No inheritance or composition

---

## Implementation Details

### Method Categories

**Initialization** (1 method):
- start(configuration)

**Account Management** (4 methods):
- createAccount(configuration)
- registerAccount(account, renew)
- deleteAccount(account)
- replaceAccount(account, configuration) - NOT IMPLEMENTED

**Call Operations** (15+ methods):
- makeCall(), answerCall(), hangupCall(), declineCall()
- holdCall(), unholdCall()
- muteCall(), unMuteCall()
- useSpeaker(), useEarpiece()
- xferCall(), xferReplacesCall(), redirectCall()
- dtmfCall()

**Messaging** (2 methods):
- sendMessage(), imTyping()

**Configuration** (5 methods):
- updateStunServers(), changeNetworkConfiguration()
- changeServiceConfiguration(), changeCodecSettings()
- changeOrientation()

**Audio Session** (2 methods):
- activateAudioSession(), deactivateAudioSession()

**Event Handlers** (7 methods):
- _onRegistrationChanged()
- _onCallReceived(), _onCallChanged(), _onCallTerminated()
- _onCallScreenLocked()
- _onMessageReceived()
- _onConnectivityChanged()

**Utility** (1 method):
- _normalize()

### File Size Breakdown

```
Endpoint.js: ~580 lines
├── Constructor: ~15 lines
├── start(): ~25 lines
├── Account methods: ~50 lines
├── Call methods: ~200 lines
├── Message methods: ~30 lines
├── Configuration: ~60 lines
├── Audio session: ~20 lines
├── Event handlers: ~50 lines
├── Utility (_normalize): ~20 lines
├── JSDoc comments: ~100 lines
└── Blank lines: ~10 lines
```

---

## Compliance

**This decision is CONSTRAINING.**

It constrains:
- All SIP operations must go through Endpoint
- Application code must hold Endpoint reference
- No direct native module access
- Event handling centralized

---

## Related Decisions

- **ADR-001**: EventEmitter Choice (Endpoint extends EventEmitter)
- **ADR-002**: Promise Pattern (all methods return Promises)
- **ADR-003**: Immutability Strategy (Endpoint creates immutable instances)

---

## Refactoring Opportunities

If Endpoint becomes too complex, consider:

### Option 1: Facade Pattern
```javascript
class Endpoint {
  constructor() {
    this.accounts = new AccountManager(this);
    this.calls = new CallManager(this);
    this.messages = new MessageManager(this);
  }
}

// Usage
endpoint.accounts.create(config);
endpoint.calls.make(account, destination);
```

### Option 2: Module Separation
```javascript
class Endpoint {
  constructor() {
    this.accountModule = new AccountModule(nativeModule);
    this.callModule = new CallModule(nativeModule);
  }
}
```

### Option 3: Keep as Is
- Document complexity
- Add comprehensive tests
- Ensure good code organization

**Recommendation**: Option 3 (keep as is) for current scope. Refactor if file exceeds 1000 lines.

---

## Notes

**Legacy Observation**: One method (replaceAccount) throws "Not implemented" error. This suggests incomplete implementation or removed functionality.

**Code Quality**: File is well-organized with clear sections, but lacks:
- Unit tests
- Error handling consistency
- Cleanup/teardown mechanism

---

*Generated by /legacy analysis*
