# Test Requirements: Call State and Duration Tracking

> Test-Driven Development flow for Call class.

**Status**: DRAFT  
**Generated**: 2026-03-04  
**Source**: Legacy analysis of src/Call.js

---

## Test Suite Overview

**Module**: Call.js  
**Purpose**: Verify call state tracking, duration calculation, and URI parsing  
**Criticality**: HIGH - Correctness-critical for billing and user experience

---

## Test Cases

### Suite: Constructor - URI Parsing

#### TC-001: Parse SIP URI with name

**Given**: SIP URI with name format `"John Doe" <sip:100@pbx.com>`  
**When**: Call object created  
**Then**: 
- `getRemoteName()` returns `"John Doe"`
- `getRemoteNumber()` returns `"100"`

**Test**:
```javascript
const call = new Call({
  remoteUri: '"John Doe" <sip:100@pbx.com>',
  // ... other required fields
});
expect(call.getRemoteName()).toBe('John Doe');
expect(call.getRemoteNumber()).toBe('100');
```

---

#### TC-002: Parse SIP URI without name

**Given**: SIP URI without name `sip:100@pbx.com`  
**When**: Call object created  
**Then**: 
- `getRemoteName()` returns `null`
- `getRemoteNumber()` returns `"100"`

**Test**:
```javascript
const call = new Call({
  remoteUri: 'sip:100@pbx.com',
  // ... other required fields
});
expect(call.getRemoteName()).toBeNull();
expect(call.getRemoteNumber()).toBe('100');
```

---

#### TC-003: Parse tel: URI

**Given**: tel: URI format `tel:+1234567890`  
**When**: Call object created with localUri  
**Then**: 
- `getLocalNumber()` returns decoded tel: URI value

**Test**:
```javascript
const call = new Call({
  localUri: 'tel:+1234567890',
  // ... other required fields
});
expect(call.getLocalNumber()).toBe('+1234567890');
```

---

#### TC-004: Handle null URI

**Given**: `remoteUri` is `null` or `undefined`  
**When**: Call object created  
**Then**: 
- `getRemoteName()` returns `null`
- `getRemoteNumber()` returns `null`

**Test**:
```javascript
const call = new Call({
  remoteUri: null,
  // ... other required fields
});
expect(call.getRemoteName()).toBeNull();
expect(call.getRemoteNumber()).toBeNull();
```

---

### Suite: Duration Tracking

#### TC-010: Initial duration

**Given**: Call just created with `totalDuration: 120`, `connectDuration: 60`  
**When**: `getTotalDuration()` called immediately  
**Then**: Returns value >= 120 (may be slightly higher due to time passage)

**Test**:
```javascript
const call = new Call({
  totalDuration: 120,
  connectDuration: 60,
  // ... other required fields
});
const duration = call.getTotalDuration();
expect(duration).toBeGreaterThanOrEqual(120);
expect(duration).toBeLessThan(125); // Allow 5 second tolerance
```

---

#### TC-011: Duration increases over time

**Given**: Call created 10 seconds ago  
**When**: `getTotalDuration()` called  
**Then**: Returns initial + elapsed time

**Test**:
```javascript
// Mock Date
const originalDate = Date;
Date = class extends originalDate {
  constructor() {
    super();
    return new originalDate(originalDate.now() + 10000); // +10 seconds
  }
};

const call = new Call({
  totalDuration: 120,
  connectDuration: 60,
  // ... other required fields
});
expect(call.getTotalDuration()).toBeGreaterThanOrEqual(130);

Date = originalDate; // Restore
```

---

#### TC-012: Connect duration when not established

**Given**: Call with `connectDuration: -1` (not established)  
**When**: `getConnectDuration()` called  
**Then**: Returns -1

**Test**:
```javascript
const call = new Call({
  totalDuration: 120,
  connectDuration: -1,
  state: 'PJSIP_INV_STATE_CALLING',
  // ... other required fields
});
expect(call.getConnectDuration()).toBe(-1);
```

---

#### TC-013: Connect duration when disconnected

**Given**: Call with state `PJSIP_INV_STATE_DISCONNECTED`  
**When**: `getConnectDuration()` called  
**Then**: Returns stored value without time compensation

**Test**:
```javascript
const call = new Call({
  totalDuration: 120,
  connectDuration: 60,
  state: 'PJSIP_INV_STATE_DISCONNECTED',
  // ... other required fields
});
expect(call.getConnectDuration()).toBe(60); // No time offset
```

---

#### TC-014: Format duration MM:SS

**Given**: Duration 125 seconds  
**When**: `_formatTime(125)` called  
**Then**: Returns `"02:05"`

**Test**:
```javascript
// Access private method via prototype or create testable instance
const call = new Call({ /* ... */ });
expect(call._formatTime(125)).toBe('02:05');
```

---

#### TC-015: Format duration with hours

**Given**: Duration 3661 seconds (1 hour, 1 minute, 1 second)  
**When**: `_formatTime(3661)` called  
**Then**: Returns `"01:01:01"`

**Test**:
```javascript
const call = new Call({ /* ... */ });
expect(call._formatTime(3661)).toBe('01:01:01');
```

---

#### TC-016: Format invalid duration

**Given**: Negative or NaN duration  
**When**: `_formatTime(-1)` or `_formatTime(NaN)` called  
**Then**: Returns `"00:00"`

**Test**:
```javascript
const call = new Call({ /* ... */ });
expect(call._formatTime(-1)).toBe('00:00');
expect(call._formatTime(NaN)).toBe('00:00');
```

---

### Suite: Call State

#### TC-020: Check held state

**Given**: Call with `held: true`  
**When**: `isHeld()` called  
**Then**: Returns `true`

---

#### TC-021: Check muted state

**Given**: Call with `muted: true`  
**When**: `isMuted()` called  
**Then**: Returns `true`

---

#### TC-022: Check speaker state

**Given**: Call with `speaker: true`  
**When**: `isSpeaker()` called  
**Then**: Returns `true`

---

#### TC-023: Check terminated state

**Given**: Call with `state: 'PJSIP_INV_STATE_DISCONNECTED'`  
**When**: `isTerminated()` called  
**Then**: Returns `true`

---

#### TC-024: Check not terminated state

**Given**: Call with `state: 'PJSIP_INV_STATE_CONNECTING'`  
**When**: `isTerminated()` called  
**Then**: Returns `false`

---

### Suite: SIP Status Codes

#### TC-030: Last status code

**Given**: Call with `lastStatusCode: 200`  
**When**: `getLastStatusCode()` called  
**Then**: Returns `200`

---

#### TC-031: Last reason phrase

**Given**: Call with `lastReason: "OK"`  
**When**: `getLastReason()` called  
**Then**: Returns `"OK"`

---

### Suite: Media Information

#### TC-040: Remote audio count

**Given**: Call with `remoteAudioCount: 1`  
**When**: `getRemoteAudioCount()` called  
**Then**: Returns `1`

---

#### TC-041: Remote video count

**Given**: Call with `remoteVideoCount: 1`  
**When**: `getRemoteVideoCount()` called  
**Then**: Returns `1`

---

#### TC-042: Audio count zero (disabled)

**Given**: Call with `audioCount: 0`  
**When**: `getAudioCount()` called  
**Then**: Returns `0` (audio disabled)

---

#### TC-043: Video count zero (disabled)

**Given**: Call with `videoCount: 0`  
**When**: `getVideoCount()` called  
**Then**: Returns `0` (video disabled)

---

### Suite: Formatted Output

#### TC-050: Get formatted remote number with name

**Given**: Call with remote name and number  
**When**: `getRemoteFormattedNumber()` called  
**Then**: Returns `"Name <Number>"`

**Test**:
```javascript
const call = new Call({
  remoteUri: '"John Doe" <sip:100@pbx.com>',
  // ... other required fields
});
expect(call.getRemoteFormattedNumber()).toBe('John Doe <100>');
```

---

#### TC-051: Get formatted remote number without name

**Given**: Call with number but no name  
**When**: `getRemoteFormattedNumber()` called  
**Then**: Returns `"Number"`

---

#### TC-052: Get formatted remote number fallback

**Given**: Call with no name and no number  
**When**: `getRemoteFormattedNumber()` called  
**Then**: Returns `remoteUri`

---

## Edge Cases

### EC-001: Rapid duration calls

**Scenario**: Multiple `getTotalDuration()` calls in rapid succession  
**Expected**: Each call returns incrementally increasing value

---

### EC-002: Time zone changes

**Scenario**: System clock changes during call  
**Expected**: Duration calculation uses `Date.getTime()` (UTC-based)

---

### EC-003: Very long calls

**Scenario**: Call lasting > 24 hours  
**Expected**: `getFormattedTotalDuration()` returns `"HH:MM:SS"` format

**Test**:
```javascript
const call = new Call({ /* ... */ });
// Simulate 25 hours
expect(call._formatTime(90000)).toBe('25:00:00');
```

---

## Performance Requirements

- **P-001**: Duration calculation SHALL complete in < 1ms
- **P-002**: URI parsing SHALL complete in < 1ms
- **P-003**: Call object creation SHALL complete in < 10ms

---

## Test Data

### Sample Call Objects

```javascript
// Incoming call example
const incomingCall = new Call({
  id: 1,
  callId: 'abc123@pbx.com',
  accountId: 0,
  localContact: '"Support" <sip:100@pbx.com>',
  localUri: 'sip:100@pbx.com',
  remoteContact: '"John Doe" <sip:200@pbx.com>',
  remoteUri: '"John Doe" <sip:200@pbx.com>',
  state: 'PJSIP_INV_STATE_INCOMING',
  stateText: 'Incoming',
  held: false,
  muted: false,
  speaker: false,
  connectDuration: -1,
  totalDuration: 0,
  remoteOfferer: false,
  remoteAudioCount: 1,
  remoteVideoCount: 0,
  audioCount: 1,
  videoCount: 0,
  lastStatusCode: null,
  lastReason: null,
  media: [],
  provisionalMedia: []
});

// Established call example
const establishedCall = new Call({
  id: 2,
  callId: 'def456@pbx.com',
  accountId: 0,
  localContact: '"Support" <sip:100@pbx.com>',
  localUri: 'sip:100@pbx.com',
  remoteContact: '"Jane Smith" <sip:300@pbx.com>',
  remoteUri: '"Jane Smith" <sip:300@pbx.com>',
  state: 'PJSIP_INV_STATE_CONFIRMED',
  stateText: 'Connected',
  held: false,
  muted: true,
  speaker: true,
  connectDuration: 300,
  totalDuration: 315,
  remoteOfferer: true,
  remoteAudioCount: 1,
  remoteVideoCount: 1,
  audioCount: 1,
  videoCount: 1,
  lastStatusCode: 200,
  lastReason: 'OK',
  media: [{type: 'audio', state: 'active'}, {type: 'video', state: 'active'}],
  provisionalMedia: []
});
```

---

*Generated by /legacy analysis*
