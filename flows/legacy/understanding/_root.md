# Understanding: Project Root

> Entry point for recursive understanding. Children are top-level logical domains.

## Phase: SPAWNING

## Hypothesis

**Initial**: React Native SIP library with account management, call handling, messaging, and video support.

## Sources

> Files/directories that inform this understanding (NOT a mirror - selective)

- **src/Endpoint.js** - Main entry point, event emitter, native module bridge initialization
- **src/Account.js** - Account configuration and state wrapper
- **src/AccountRegistration.js** - Registration status tracking
- **src/Call.js** - Call state and operations (hold, transfer, DTMF, etc.)
- **src/Message.js** - SIP instant messaging
- **src/PreviewVideoView.js** - Local video preview component
- **src/RemoteVideoView.js** - Remote party video component
- **index.js** - Public API exports

## Validated Understanding

> Updated during EXPLORING phase

**Core Architecture**:
1. **Endpoint** is the central event emitter and coordinator
   - Initializes native PjSIP module via `NativeModules.PjSipModule`
   - Subscribes to 7 native events: registration, call lifecycle, messaging, connectivity
   - Manages accounts and calls collections
   - All operations return Promises with callback-based native module communication

2. **Account** is a data wrapper with no behavior
   - Holds account configuration (username, domain, password, proxy, transport)
   - Contains AccountRegistration for status tracking
   - Pure getter methods, no business logic

3. **Call** is a rich state holder with time-tracking
   - Parses SIP URIs to extract name/number
   - Tracks call duration with local time offset compensation
   - Exposes SIP call state machine (NULL, CALLING, INCOMING, EARLY, CONNECTING, CONFIRMED, DISCONNECTED)
   - No behavior - all operations performed by Endpoint

4. **Message** is a simple data wrapper
   - Parses From URI for name/number extraction
   - Holds message body and content type

5. **Video Views** are thin React Native wrappers
   - requireNativeComponent for native UI components
   - PropTypes validation for deviceId/windowId and objectFit

**Key Patterns**:
- Promise-based async API with callback pattern for native calls
- Event-driven architecture for state changes
- Data objects (Account, Call, Message) are immutable snapshots
- Endpoint is the only active component with behavior

## Children Identified

> Deeper concepts spawned during SPAWNING phase

| Child | Hypothesis | Status |
|-------|------------|--------|
| endpoint-core | Endpoint initialization, native module bridge, event subscription | PENDING |
| account-management | Account CRUD operations, registration lifecycle | PENDING |
| call-handling | Call state machine, operations (make/answer/hold/transfer) | PENDING |
| messaging | IM messaging, typing notifications | PENDING |
| video-components | Native video view integration | PENDING |
| event-system | Native-to-JS event routing and emission | PENDING |

## Dependencies

- **Uses**: React Native NativeModules, DeviceEventEmitter, events.EventEmitter
- **Used by**: React Native applications integrating SIP/VoIP

## Key Insights

1. **Separation of Concerns**: Endpoint handles all behavior, data classes are passive
2. **Native Bridge Pattern**: All operations delegate to NativeModules.PjSipModule
3. **Event-Driven State**: State changes flow from native to JS via DeviceEventEmitter
4. **Time Compensation**: Call duration tracks construction time to compute real-time duration
5. **SIP URI Parsing**: Repeated pattern of extracting name/number from SIP URIs

## ADR Candidates

- **EventEmitter Choice**: Using Node.js 'events' module vs React Native patterns
- **Promise vs Callback**: Promise wrapper over callback-based native module API
- **Data Immutability**: Account/Call/Message as immutable snapshots vs observable state
- **Video Component Architecture**: Separate components for preview vs remote streams

## Flow Recommendation

**Primary Flow**: SDD (Spec-Driven Development)
- **Type**: SDD
- **Confidence**: high
- **Rationale**: Internal service library, no stakeholder-facing documentation needed, behavior defined by PjSIP specification

**Secondary Flows**:
- **TDD** for call state machine logic (correctness-critical)
- **DDD** for public API documentation (developer-facing)

## Children Spawned

```
[endpoint-core, account-management, call-handling, messaging, video-components, event-system]
```

## Synthesis

> Updated after all children complete

### Complete System Understanding

**react-native-sip2** is a React Native library providing SIP/VoIP functionality via PjSIP native bridge.

**Architecture Summary**:

```
┌─────────────────────────────────────────┐
│         Application Code                │
│    (listens to Endpoint events)         │
└───────────────┬─────────────────────────┘
                │
┌───────────────▼─────────────────────────┐
│  Endpoint (EventEmitter + Coordinator)  │
│  - start(), createAccount(), makeCall() │
│  - 40+ public methods                   │
│  - 7 native event subscriptions         │
└───────────────┬─────────────────────────┘
                │
┌───────────────▼─────────────────────────┐
│  DeviceEventEmitter (React Native)      │
│  - pjSipRegistrationChanged             │
│  - pjSipCallReceived/Changed/Terminated │
│  - pjSipMessageReceived                 │
│  - pjSipConnectivityChanged             │
└───────────────┬─────────────────────────┘
                │
┌───────────────▼─────────────────────────┐
│  NativeModules.PjSipModule              │
│  (Native PjSIP implementation)          │
│  - iOS/Android native code              │
└─────────────────────────────────────────┘
```

**Data Flow**:
1. **Commands**: App -> Endpoint -> NativeModules (Promise-based)
2. **Events**: Native -> DeviceEventEmitter -> Endpoint -> App (EventEmitter)
3. **Data**: Immutable snapshots (Account, Call, Message)

**Key Characteristics**:
- **God Object Pattern**: Endpoint handles all behavior
- **Immutable Data**: Account/Call/Message are snapshots, no setters
- **Promise Wrapper**: Callback-to-Promise over native callbacks
- **Event Translation**: Native events -> class instances -> app
- **No Error Handling**: Raw data rejections, no Error objects
- **Time Compensation**: Call duration tracks construction time

**Code Quality Observations**:
- 580+ lines in Endpoint.js (high complexity)
- One unimplemented method (replaceAccount)
- Multiple TODO comments
- No visible unit tests
- Memory leak risk (event listeners never removed)

### From Children

| Domain | Key Insight | Flow Type |
|--------|-------------|-----------|
| endpoint-core | God object, 40+ methods, no error handling | SDD |
| account-management | Immutable snapshots, composition pattern | SDD |
| call-handling | Time compensation, SIP state machine | TDD |
| messaging | Simple data wrapper, URI parsing | SDD |
| video-components | Thin native wrappers, PropTypes only | VDD |
| event-system | Dual event system, no cleanup | SDD |

## Bubble Up

> Summary to pass to parent during EXITING

- Complete SIP library with 6 domains
- Endpoint: central coordinator, event-driven
- Immutable data objects, Promise-based API
- Native bridge via DeviceEventEmitter
- Candidates: 4 SDD flows, 1 TDD flow, 1 VDD flow
- ADRs: EventEmitter choice, error handling, immutability

---

*Phase: SYNTHESIZING | Depth: 0 | Parent: none*
