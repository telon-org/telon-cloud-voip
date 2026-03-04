# Understanding: Endpoint Core

> Central SIP endpoint initialization, native module bridge, and event coordination.

## Phase: EXPLORING

## Hypothesis

Endpoint is the main entry point that:
- Initializes the native PjSIP module
- Subscribes to native events
- Provides Promise-based API for all SIP operations
- Manages collections of accounts and calls

## Sources

- **src/Endpoint.js** - Main endpoint implementation (580+ lines)
- **index.js** - Public API export

## Validated Understanding

> Updated during EXPLORING phase

**Constructor & Initialization**:
```javascript
constructor() {
    super(); // Inherits from EventEmitter
    // Subscribe to 7 native events:
    DeviceEventEmitter.addListener('pjSipRegistrationChanged', ...)
    DeviceEventEmitter.addListener('pjSipCallReceived', ...)
    DeviceEventEmitter.addListener('pjSipCallChanged', ...)
    DeviceEventEmitter.addListener('pjSipCallTerminated', ...)
    DeviceEventEmitter.addListener('pjSipCallScreenLocked', ...)
    DeviceEventEmitter.addListener('pjSipMessageReceived', ...)
    DeviceEventEmitter.addListener('pjSipConnectivityChanged', ...)
}
```

**Core Methods**:

1. **start(configuration)** - Initializes PjSIP module
   - Returns Promise with accounts[], calls[], and extra data
   - Instantiates Account and Call objects from native data

2. **Account Management**:
   - `createAccount(configuration)` - Add account, starts registration
   - `registerAccount(account, renew)` - Update registration
   - `deleteAccount(account)` - Unregister and remove
   - `replaceAccount()` - NOT IMPLEMENTED (throws error)

3. **Call Operations**:
   - `makeCall(account, destination, callSettings, msgData)` - Outgoing call
   - `answerCall(call)` - Answer incoming
   - `hangupCall(call)` - Terminate call
   - `declineCall(call)` - Reject with 603
   - `holdCall(call)` / `unholdCall(call)` - Hold/resume
   - `muteCall(call)` / `unMuteCall(call)` - Mute/unmute
   - `useSpeaker(call)` / `useEarpiece(call)` - Audio routing
   - `xferCall(account, call, destination)` - Blind transfer
   - `xferReplacesCall(call, destCall)` - Attended transfer
   - `redirectCall(account, call, destination)` - Forward call
   - `dtmfCall(call, digits)` - Send DTMF tones

4. **Messaging**:
   - `sendMessage(account, destination, msg)` - Send IM
   - `imTyping(account, destination, isTyping)` - Typing indicator

5. **Configuration**:
   - `updateStunServers(accountId, stunServerList)`
   - `changeNetworkConfiguration(configuration)`
   - `changeServiceConfiguration(configuration)`
   - `changeCodecSettings(codecSettings)`
   - `changeOrientation(orientation)` - Video orientation

6. **Audio Session**:
   - `activateAudioSession()` / `deactivateAudioSession()`

**Event Handlers** (private):
- `_onRegistrationChanged(account)` -> emits `registration_changed`
- `_onCallReceived(call)` -> emits `call_received`
- `_onCallChanged(call)` -> emits `call_changed`
- `_onCallTerminated(call)` -> emits `call_terminated`
- `_onCallScreenLocked(lock)` -> emits `call_screen_locked`
- `_onMessageReceived(message)` -> emits `message_received`
- `_onConnectivityChanged(available)` -> emits `connectivity_changed`

**Utility**:
- `_normalize(account, destination)` - Converts destination to SIP URI format

**Pattern Analysis**:
- All native calls follow: `NativeModules.PjSipModule.method(args, (successful, data) => {...})`
- Success: resolve with data/objects
- Failure: reject with data (error message)
- No retry logic, no error classification

## Children Identified

> Deeper concepts spawned during SPAWNING phase

| Child | Hypothesis | Status |
|-------|------------|--------|
| initialization | start() method, native module bootstrap | PENDING |
| event-routing | Event listener registration and emission | PENDING |
| promise-bridge | Promise wrapper over native callbacks | PENDING |
| call-operations | All call-related methods | PENDING |
| account-operations | Account CRUD and registration | PENDING |

## Dependencies

- **Uses**: 
  - React Native (DeviceEventEmitter, NativeModules)
  - events.EventEmitter (Node.js event emitter)
  - Account, Call, Message classes
- **Used by**: Application code integrating SIP

## Key Insights

1. **God Object Pattern**: Endpoint handles everything - initialization, accounts, calls, messaging, configuration
2. **EventEmitter Hybrid**: Extends EventEmitter but also uses DeviceEventEmitter for native events
3. **No Error Handling**: No try/catch, no error classification, all rejections are raw data
4. **SIP URI Normalization**: Automatic conversion of bare numbers to `sip:number @realm` format
5. **Incomplete Implementation**: `replaceAccount()` throws "Not implemented"
6. **TODO Comments**: Multiple TODOs in event handlers for documentation

## ADR Candidates

- **EventEmitter Choice**: Why Node.js EventEmitter instead of React Native patterns?
- **Promise Pattern**: Callback-to-Promise wrapper without async/await
- **Error Handling Strategy**: Raw data rejection vs Error objects
- **God Object vs Modularization**: All functionality in single class

## Flow Recommendation

- **Type**: SDD
- **Confidence**: high
- **Rationale**: Internal service logic, well-defined behavior by PjSIP spec, no stakeholder docs needed

## Children Spawned

```
[initialization, event-routing, promise-bridge, call-operations, account-operations]
```

## Synthesis

> Updated after all children complete

**Decision**: No deeper recursion needed. Endpoint-core is well-understood at this level.

### Combined Understanding

Endpoint is the **single entry point** for all SIP functionality:

**Responsibilities**:
1. Native module initialization and lifecycle
2. Event routing (native -> EventEmitter -> application)
3. Account management (create, register, delete)
4. Call operations (make, answer, hold, transfer, DTMF, etc.)
5. Messaging (send IM, typing indicators)
6. Configuration (network, codec, orientation)

**Design Characteristics**:
- God object pattern - all behavior centralized
- Promise-based async API over callback native bridge
- No error handling or retry logic
- Immutable data objects (Account, Call, Message)
- Automatic SIP URI normalization

**Code Quality Notes**:
- 580+ lines in single file
- One unimplemented method (replaceAccount)
- Multiple TODO comments for documentation
- No unit tests visible in source

## Bubble Up

> Summary to pass to parent during EXITING

- Endpoint is central coordinator with 40+ public methods
- All native communication via Promise-wrapped callbacks
- 7 native events mapped to public EventEmitter events
- Handles accounts, calls, messaging, configuration
- No error handling beyond Promise rejections
- Candidate for SDD flow (internal service logic)

---

*Phase: SYNTHESIZING | Depth: 1 | Parent: root*
