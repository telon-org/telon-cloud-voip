# Understanding: Event System

> Native-to-JavaScript event routing and emission architecture.

## Phase: EXPLORING

## Sources

- **src/Endpoint.js** - Event listener subscriptions and emissions

## Validated Understanding

**Event Flow Architecture**:

```
Native Module (PjSipModule)
    ↓ DeviceEventEmitter
Endpoint (subscribes to 7 native events)
    ↓ EventEmitter (Node.js)
Application (listens to Endpoint events)
```

**Native Events** (subscribed in constructor):

| Native Event | Emitted As | Data Type |
|--------------|-----------|-----------|
| `pjSipRegistrationChanged` | `registration_changed` | Account |
| `pjSipCallReceived` | `call_received` | Call |
| `pjSipCallChanged` | `call_changed` | Call |
| `pjSipCallTerminated` | `call_terminated` | Call |
| `pjSipCallScreenLocked` | `call_screen_locked` | boolean |
| `pjSipMessageReceived` | `message_received` | Message |
| `pjSipConnectivityChanged` | `connectivity_changed` | Account\|boolean |

**Event Handlers** (private methods):
- `_onRegistrationChanged(data)` -> `emit("registration_changed", new Account(data))`
- `_onCallReceived(data)` -> `emit("call_received", new Call(data))`
- `_onCallChanged(data)` -> `emit("call_changed", new Call(data))`
- `_onCallTerminated(data)` -> `emit("call_terminated", new Call(data))`
- `_onCallScreenLocked(lock)` -> `emit("call_screen_locked", lock)`
- `_onMessageReceived(data)` -> `emit("message_received", new Message(data))`
- `_onConnectivityChanged(available)` -> `emit("connectivity_changed", available)`

**EventEmitter Pattern**:
- Endpoint extends Node.js `EventEmitter`
- Uses React Native `DeviceEventEmitter` for native events
- Dual event system: native bridge + application API

**Usage Pattern**:
```javascript
const endpoint = new Endpoint();
endpoint.on('registration_changed', (account) => {...});
endpoint.on('call_received', (call) => {...});
endpoint.on('call_changed', (call) => {...});
endpoint.on('call_terminated', (call) => {...});
endpoint.on('message_received', (message) => {...});
```

**Key Insights**:
1. **Event Translation**: Native events wrapped in Node.js EventEmitter
2. **Data Transformation**: Raw native data -> Class instances (Account, Call, Message)
3. **No Event Filtering**: All events passed through, no filtering or throttling
4. **No Error Events**: No error event channel, errors only via Promise rejections
5. **Memory Leak Risk**: Event listeners never removed (no cleanup in code)

## Flow Recommendation

- **Type**: SDD
- **Confidence**: high
- **Rationale**: Internal event routing, service infrastructure

## Bubble Up

- Dual event system: DeviceEventEmitter + EventEmitter
- 7 native events translated to public API
- Data transformation: raw -> class instances
- No cleanup/teardown visible
- No error event channel

---

*Phase: EXPLORING | Depth: 1 | Parent: root*
