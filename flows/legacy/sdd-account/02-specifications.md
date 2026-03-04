# Specifications: Account Management

> Technical specifications for Account and AccountRegistration implementation.

**Status**: DRAFT  
**Generated**: 2026-03-04  
**Source**: Legacy analysis of src/Account.js, src/AccountRegistration.js

---

## Class Specifications

### Account Class

**Purpose**: Immutable data wrapper for SIP account configuration.

**Constructor**:
```javascript
constructor(data: Object)
```

**Constructor Input**:
```javascript
{
  id: number,              // Account ID from PjSIP
  uri: string,             // SIP URI for registration
  name: string,            // Full name
  username: string,        // SIP username
  domain: string|null,     // Domain (may be null)
  password: string,        // Password (plaintext)
  proxy: string,           // Proxy server
  transport: string,       // TCP | UDP
  contactParams: string,   // Contact header parameters
  contactUriParams: string, // Contact URI parameters
  regServer: string,       // Registration server
  regTimeout: number,      // Registration timeout (seconds)
  regContactParams: string, // Registration contact parameters
  regHeaders: object,      // Registration headers
  registration: {          // Registration status data
    status: string|null,
    statusText: string|null,
    active: boolean,
    reason: string|null
  }
}
```

**Internal State**:
```javascript
this._data = data;  // Store entire data object
this._registration = new AccountRegistration(data['registration']);
```

**Methods** (all getters):

| Method | Returns | Implementation |
|--------|---------|----------------|
| `getId()` | number | `return this._data.id` |
| `getURI()` | string | `return this._data.uri` |
| `getName()` | string | `return this._data.name` |
| `getUsername()` | string | `return this._data.username` |
| `getDomain()` | string\|null | `return this._data.domain` |
| `getPassword()` | string | `return this._data.password` |
| `getProxy()` | string | `return this._data.proxy` |
| `getTransport()` | string | `return this._data.transport` |
| `getContactParams()` | string | `return this._data.contactParams` |
| `getContactUriParams()` | string | `return this._data.contactUriParams` |
| `getRegServer()` | string | `return this._data.regServer \|\| ""` |
| `getRegTimeout()` | number | `return this._data.regTimeout` |
| `getRegContactParams()` | string | `return this._data.regContactParams` |
| `getRegHeaders()` | object | `return this._data.regHeaders` |
| `getRegistration()` | AccountRegistration | `return this._registration` |

**Special Behavior**:
- `getRegServer()`: Returns empty string if null/undefined (defensive)
- All other getters: Direct property access (no validation)

---

### AccountRegistration Class

**Purpose**: Immutable data wrapper for registration status.

**Constructor**:
```javascript
constructor({
  status,
  statusText,
  active,
  reason
}: {
  status: string|null,
  statusText: string|null,
  active: boolean,
  reason: string|null
})
```

**Internal State**:
```javascript
this._status = status;
this._statusText = statusText;
this._active = active;
this._reason = reason;
```

**Methods**:

| Method | Returns | Implementation |
|--------|---------|----------------|
| `getStatus()` | string\|null | `return this._status` |
| `getStatusText()` | string\|null | `return this._statusText` |
| `isActive()` | boolean | `return this._active` |
| `getReason()` | string\|null | `return this._reason` |
| `toJson()` | object | See below |

**toJson() Implementation**:
```javascript
toJson() {
  return {
    status: this._status,
    statusText: this._statusText,
    active: this._active,
    reason: this._reason
  }
}
```

---

## Data Flow

### Account Creation Flow

```
Native Module (createAccount)
    ↓ (callback with data)
Endpoint.createAccount()
    ↓
new Account(data)
    ├─> Store data in this._data
    └─> new AccountRegistration(data.registration)
            ├─> Store status
            ├─> Store statusText
            ├─> Store active
            └─> Store reason
    ↓
Resolve Promise with Account instance
```

### Registration Update Flow

```
Native Module (registration changed)
    ↓ (DeviceEventEmitter event)
Endpoint._onRegistrationChanged(data)
    ↓
new Account(data)  // New instance with updated registration
    ↓
emit('registration_changed', account)
    ↓
Application receives updated Account
```

---

## Immutability Pattern

**Design Decision**: Snapshots, not observables.

**Rationale**:
- Simple mental model
- No mutation bugs
- Easy to serialize
- Matches native callback pattern

**Trade-offs**:
- (+) Predictable behavior
- (+) No accidental mutations
- (-) New object on every state change
- (-) No change tracking within object

**Example**:
```javascript
// Initial account
const account1 = new Account({id: 1, name: "John", registration: {active: false, ...}});

// After registration
const account2 = new Account({id: 1, name: "John", registration: {active: true, ...}});

// account1 !== account2 (different instances)
// account1.getRegistration().isActive() === false
// account2.getRegistration().isActive() === true
```

---

## Null Handling

### Account

| Property | Nullable | Default |
|----------|----------|---------|
| domain | Yes | null |
| regServer | Yes | "" (empty string in getter) |

### AccountRegistration

| Property | Nullable | Meaning |
|----------|----------|---------|
| status | Yes | Not registered if null |
| statusText | Yes | No status text available |
| reason | Yes | No reason phrase |

---

## Security Considerations

### Password Storage

**Current**: Password stored in plaintext in `this._data.password`

**Implications**:
- Accessible via `account.getPassword()`
- Visible in memory dumps
- Visible in debugger
- Serialized if account object is logged

**Mitigation** (not implemented):
- Store password in secure storage (Keychain/Keystore)
- Clear password after authentication
- Use token-based authentication

---

## Usage Examples

### Access Account Properties

```javascript
const account = await endpoint.createAccount({
  name: "John Doe",
  username: "100",
  domain: "pbx.com",
  password: "secret"
});

console.log(account.getId());        // 1
console.log(account.getName());      // "John Doe"
console.log(account.getUsername());  // "100"
console.log(account.getDomain());    // "pbx.com"
console.log(account.getRegServer()); // "" (empty string)
```

### Check Registration Status

```javascript
const registration = account.getRegistration();
console.log(registration.getStatus());      // 200 (or null)
console.log(registration.getStatusText());  // "OK" (or null)
console.log(registration.isActive());       // true/false
console.log(registration.getReason());      // Reason phrase (or null)
```

### Serialize Registration

```javascript
const json = account.getRegistration().toJson();
// {status: 200, statusText: "OK", active: true, reason: ""}
```

---

## Testing Considerations

### Unit Tests

- Test all getters return correct values
- Test `getRegServer()` returns empty string for null
- Test AccountRegistration.toJson() output
- Test immutability (no setters exist)

### Integration Tests

- Test account creation via Endpoint
- Test registration status updates
- Test account serialization/deserialization

---

*Generated by /legacy analysis*
