# Specifications: SIP Endpoint Core

> Technical specifications for Endpoint implementation.

**Status**: DRAFT  
**Generated**: 2026-03-04  
**Source**: Legacy analysis of src/Endpoint.js

---

## Architecture

### Component Diagram

```
┌──────────────────────────────────────────────────────────┐
│                    Application Code                       │
│  endpoint.on('call_received', (call) => handle(call))    │
└────────────────────┬─────────────────────────────────────┘
                     │ EventEmitter
┌────────────────────▼─────────────────────────────────────┐
│                     Endpoint                             │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Constructor: Subscribe to 7 native events         │  │
│  │  - pjSipRegistrationChanged                        │  │
│  │  - pjSipCallReceived/Changed/Terminated            │  │
│  │  - pjSipCallScreenLocked                           │  │
│  │  - pjSipMessageReceived                            │  │
│  │  - pjSipConnectivityChanged                        │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Public API (40+ methods)                          │  │
│  │  - start(), createAccount(), makeCall(), ...       │  │
│  │  - All return Promises                             │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Event Handlers (private)                          │  │
│  │  - _onRegistrationChanged(), _onCallReceived(), .. │  │
│  │  - Transform native data -> class instances        │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Utility                                           │  │
│  │  - _normalize(): SIP URI normalization             │  │
│  └────────────────────────────────────────────────────┘  │
└────────────────────┬─────────────────────────────────────┘
                     │ DeviceEventEmitter
┌────────────────────▼─────────────────────────────────────┐
│            NativeModules.PjSipModule                      │
│  (Native iOS/Android PjSIP implementation)               │
└──────────────────────────────────────────────────────────┘
```

---

## Interface Specifications

### Constructor

```javascript
constructor()
```

**Behavior**:
1. Call `super()` (EventEmitter initialization)
2. Subscribe to 7 native events via `DeviceEventEmitter.addListener()`
3. Bind event handlers to `this` context

**Side Effects**:
- Event listeners registered for lifetime of application
- No cleanup/teardown mechanism

---

### start(configuration)

```javascript
async start(configuration: Object) => Promise<{
  accounts: Account[],
  calls: Call[],
  ...extra
}>
```

**Algorithm**:
```
1. Call NativeModules.PjSipModule.start(configuration, callback)
2. In callback:
   a. If successful:
      - Create Account[] from data.accounts
      - Create Call[] from data.calls
      - Extract extra properties
      - Resolve with {accounts, calls, ...extra}
   b. If failure:
      - Reject with data
```

**Error Handling**:
- Rejection value: raw data from native (type unknown)

---

### createAccount(configuration)

```javascript
async createAccount(configuration: Object) => Promise<Account>
```

**Configuration Object**:
```javascript
{
  name: "John Doe",           // Full name
  username: "100",            // SIP username
  domain: "pbx.com",          // SIP domain
  password: "XXXXXX",         // Password (plaintext)
  proxy: "192.168.100.1:5060", // Optional proxy
  transport: "TCP",           // TCP (default) | UDP
  regServer: "pbx.com",       // Registration server (default: domain)
  regTimeout: 300             // Registration timeout (default: 300)
}
```

**Algorithm**:
```
1. Call NativeModules.PjSipModule.createAccount(config, callback)
2. If successful:
   - Create new Account(data)
   - Resolve with Account instance
3. If failure:
   - Reject with data
```

**Side Effects**:
- If registration configured, SIP REGISTER sent automatically

---

### makeCall(account, destination, callSettings, msgData)

```javascript
async makeCall(
  account: Account,
  destination: string,
  callSettings?: PjSipCallSetttings,
  msgData?: PjSipMsgData
) => Promise<Call>
```

**Type Definitions**:
```javascript
// Call settings
type PjSipCallSetttings = {
  flag: number,              // Bitmask of pjsua_call_flag constants
  req_keyframe_method: number, // Keyframe request methods allowed
  aud_cnt: number,           // Active audio streams (0 = disable audio)
  vid_cnt: number            // Active video streams (0 = disable video)
}

// Message data
type PjSipMsgData = {
  target_uri: string,        // Target URI
  hdr_list: PjSipHdrList,    // Additional headers
  content_type: string,      // MIME type of message body
  msg_body: string           // Message body
}

// Headers
type PjSipHdrList = {
  "X-Custom-Header": "Value",
  ...
}
```

**Algorithm**:
```
1. Normalize destination: destination = _normalize(account, destination)
2. Call NativeModules.PjSipModule.makeCall(account.id, destination, callSettings, msgData, callback)
3. If successful:
   - Create new Call(data)
   - Resolve with Call instance
4. If failure:
   - Reject with data
```

---

### _normalize(account, destination)

```javascript
_normalize(account: Account, destination: string) => string
```

**Algorithm**:
```
1. If destination starts with "sip:":
   - Return destination unchanged
2. Otherwise:
   - Get realm = account.getRegServer()
   - If realm empty:
     - realm = account.getDomain()
     - If realm contains ":":
       - realm = substring before ":"
   - Return "sip:" + destination + "@" + realm
```

**Examples**:
```
Input:  destination="100", account.regServer="pbx.com"
Output: "sip:100@pbx.com"

Input:  destination="100", account.domain="pbx.com:5060"
Output: "sip:100@pbx.com"

Input:  destination="sip:100@pbx.com"
Output: "sip:100@pbx.com" (unchanged)
```

---

## Event Specifications

### Event Flow

```
Native Module
    ↓ (native event with data)
DeviceEventEmitter
    ↓ (listener callback)
Endpoint._on*() method
    ↓ (transform: new Account/Call/Message(data))
EventEmitter.emit(eventName, instance)
    ↓ (application listener)
Application callback
```

### Event Mapping Table

| Native Event | Handler | Emitted Event | Data Type |
|--------------|---------|---------------|-----------|
| `pjSipRegistrationChanged` | `_onRegistrationChanged(data)` | `registration_changed` | Account |
| `pjSipCallReceived` | `_onCallReceived(data)` | `call_received` | Call |
| `pjSipCallChanged` | `_onCallChanged(data)` | `call_changed` | Call |
| `pjSipCallTerminated` | `_onCallTerminated(data)` | `call_terminated` | Call |
| `pjSipCallScreenLocked` | `_onCallScreenLocked(lock)` | `call_screen_locked` | boolean |
| `pjSipMessageReceived` | `_onMessageReceived(data)` | `message_received` | Message |
| `pjSipConnectivityChanged` | `_onConnectivityChanged(available)` | `connectivity_changed` | Account \| boolean |

---

## State Management

### Account State

Accounts are **immutable snapshots**:
- Created from native data
- Never updated in place
- New instances on state changes

**Lifecycle**:
```
createAccount() -> Account instance
    ↓
registration_changed event -> New Account instance
    ↓
deleteAccount() -> Account removed
```

### Call State

Calls are **immutable snapshots** with time compensation:

**Duration Calculation**:
```javascript
constructor() {
  this._constructionTime = Math.round(new Date().getTime() / 1000);
}

getTotalDuration() {
  let time = Math.round(new Date().getTime() / 1000);
  let offset = time - this._constructionTime;
  return this._totalDuration + offset;
}
```

**Rationale**: Native provides duration at snapshot time; JS calculates real-time duration using offset.

---

## Error Handling

### Pattern

```javascript
NativeModules.PjSipModule.method(args, (successful, data) => {
  if (successful) {
    resolve(data);  // or resolve(new Class(data))
  } else {
    reject(data);   // Raw data, not Error object
  }
});
```

**Characteristics**:
- No try/catch blocks
- No Error object creation
- Rejection value is raw native data (string or object)
- No error classification
- No retry logic

**Implications**:
- Application cannot distinguish error types
- No automatic recovery
- Error messages depend on native implementation

---

## Configuration Specifications

### changeOrientation(orientation)

```javascript
changeOrientation(orientation: string) => void
```

**Valid Values**:
- `'PJMEDIA_ORIENT_UNKNOWN'`
- `'PJMEDIA_ORIENT_ROTATE_90DEG'`
- `'PJMEDIA_ORIENT_ROTATE_270DEG'`
- `'PJMEDIA_ORIENT_ROTATE_180DEG'`
- `'PJMEDIA_ORIENT_NATURAL'`

**Validation**:
```javascript
if (orientations.indexOf(orientation) === -1) {
  throw new Error(`Invalid ${JSON.stringify(orientation)} device orientation`)
}
```

**Behavior**:
- Throws synchronously if invalid
- Calls native module if valid
- Returns void (no Promise)

---

## Dependencies

### External

| Dependency | Usage |
|------------|-------|
| `react-native` | NativeModules, DeviceEventEmitter, requireNativeComponent |
| `events` | Node.js EventEmitter class |

### Internal

| Dependency | Usage |
|------------|-------|
| `Account` | Account data representation |
| `Call` | Call state representation |
| `Message` | Message data representation |

---

## Implementation Notes

### Not Implemented

**replaceAccount(account, configuration)**:
```javascript
replaceAccount(account, configuration) {
    throw new Error("Not implemented");
}
```

**Rationale**: Unknown (legacy code)

### TODO Comments

Multiple event handlers have TODO for documentation:
```javascript
/**
 * TODO
 *
 * @event Endpoint#call_received
 * @property {Call} call
 */
```

---

*Generated by /legacy analysis*
