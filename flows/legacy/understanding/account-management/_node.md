# Understanding: Account Management

> SIP account configuration, registration lifecycle, and state management.

## Phase: EXPLORING

## Hypothesis

Account management handles:
- SIP account configuration storage
- Registration status tracking
- Account lifecycle (create, update, delete)
- No business logic - pure data wrappers

## Sources

- **src/Account.js** - Account data wrapper (100+ lines)
- **src/AccountRegistration.js** - Registration status (50 lines)

## Validated Understanding

> Updated during EXPLORING phase

**Account Class** (src/Account.js):

**Constructor**:
```javascript
constructor(data) {
    this._data = data;
    this._registration = new AccountRegistration(data['registration']);
}
```

**Properties** (all getters, no setters - immutable):
- `getId()` - Account ID from PjSIP
- `getURI()` - SIP URI for registration (e.g., "sip:serviceprovider")
- `getName()` - Full name from createAccount
- `getUsername()` - SIP username
- `getDomain()` - Domain/server address
- `getPassword()` - Password (plaintext stored)
- `getProxy()` - Proxy server
- `getTransport()` - Transport protocol (TCP/UDP)
- `getContactParams()` - Contact header parameters
- `getContactUriParams()` - Contact URI parameters
- `getRegServer()` - Registration server
- `getRegTimeout()` - Registration timeout
- `getRegContactParams()` - Registration contact parameters
- `getRegHeaders()` - Registration headers
- `getRegistration()` - AccountRegistration instance

**AccountRegistration Class** (src/AccountRegistration.js):

**Properties**:
- `getStatus()` - SIP status code (e.g., 200 OK, null if not registered)
- `getStatusText()` - Human-readable status
- `isActive()` - Boolean: currently registered?
- `getReason()` - Reason phrase from server
- `toJson()` - Serialize to plain object

**Key Patterns**:
1. **Immutable Data**: All properties are getters, no setters
2. **Composition**: Account contains AccountRegistration
3. **Snapshot Pattern**: Objects created from native data, never updated
4. **No Validation**: No type checking or validation in getters
5. **Null Handling**: Some methods return null (domain, status)

**Lifecycle** (managed by Endpoint):
1. **Create**: `Endpoint.createAccount(config)` -> Account instance
2. **Register**: `Endpoint.registerAccount(account, renew)` -> updates registration status
3. **Update**: Native events trigger new Account instances
4. **Delete**: `Endpoint.deleteAccount(account)` -> unregisters

**Registration Flow**:
```
Account Created
    ↓
Registration Started (automatic if configured)
    ↓
pjSipRegistrationChanged event
    ↓
New Account instance with updated registration
    ↓
Application receives via Endpoint#registration_changed
```

## Children Identified

> Deeper concepts spawned during SPAWNING phase

| Child | Hypothesis | Status |
|-------|------------|--------|
| account-configuration | Account properties and SIP settings | PENDING |
| registration-state | Registration status tracking | PENDING |

**Decision**: No deeper recursion needed. Account management is simple data structure.

## Dependencies

- **Uses**: AccountRegistration (composition)
- **Used by**: Endpoint (account management methods)

## Key Insights

1. **Passive Data Object**: Account has zero business logic
2. **Immutable by Convention**: No setters, data only set in constructor
3. **Registration is First-Class**: Separate class for registration status
4. **Snapshot Pattern**: New instances on state changes, not mutations
5. **Plaintext Passwords**: Password stored as-is (security consideration)
6. **No Serialization**: No toJSON on Account, only on AccountRegistration

## ADR Candidates

- **Immutability Strategy**: Why immutable snapshots vs observable state?
- **Password Storage**: Plaintext in memory (security trade-off)
- **Composition over Inheritance**: AccountRegistration as separate class

## Flow Recommendation

- **Type**: SDD
- **Confidence**: high
- **Rationale**: Internal data structures, no stakeholder-facing logic

## Synthesis

> Combined understanding

**Account Management Domain**:

**Purpose**: Represent SIP account configuration and registration state as immutable data objects.

**Characteristics**:
- Pure data wrappers with getter-only interface
- Composition pattern (Account has AccountRegistration)
- Snapshot-based state updates (new instances on changes)
- No validation, no business logic
- Serialized to/from native via plain objects

**Relationship to Endpoint**:
- Endpoint creates Account instances from native data
- Endpoint triggers registration via native calls
- Account objects passed to Endpoint for operations
- Registration updates arrive via events, create new Account instances

**Code Quality**:
- Simple, focused classes
- Clear separation of concerns
- No tests visible
- AccountRegistration.toJson() suggests serialization needs

## Bubble Up

> Summary to pass to parent during EXITING

- Account and AccountRegistration are immutable data wrappers
- No business logic - all behavior in Endpoint
- Composition pattern for registration status
- Snapshot-based state updates via events
- Plaintext password storage (security note)
- Candidate for SDD flow (internal service data structures)

---

*Phase: EXPLORING | Depth: 1 | Parent: root*
