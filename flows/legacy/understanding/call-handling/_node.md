# Understanding: Call Handling

> Call state machine, operations (make/answer/hold/transfer), and call lifecycle management.

## Phase: EXPLORING

## Sources

- **src/Call.js** - Call state and operations (400+ lines)

## Validated Understanding

**Call Class** - Rich state holder for SIP call session.

**Constructor Parsing**:
- Extracts name/number from SIP URIs using regex
- Handles formats: `"Name" <sip:number @domain>` and `sip:number @domain`
- Handles `tel:` URI scheme
- Stores construction timestamp for duration calculation

**State Properties**:
- `getId()` - Call ID
- `getAccountId()` - Parent account ID
- `getCallId()` - SIP Call-ID header
- `getState()` - SIP invite state (PJSIP_INV_STATE_*)
- `getStateText()` - Human-readable state
- `getLocalContact()`, `getLocalUri()` - Local party info
- `getRemoteContact()`, `getRemoteUri()` - Remote party info
- `getRemoteName()`, `getRemoteNumber()` - Parsed remote info
- `getRemoteFormattedNumber()` - Formatted display string

**Duration Tracking** (with time compensation):
- `getTotalDuration()` - Time since call start (updates in real-time)
- `getConnectDuration()` - Time since connected (0 if not established)
- `getFormattedTotalDuration()` - "MM:SS" or "HH:MM:SS" format
- `getFormattedConnectDuration()` - Formatted connected duration

**Call State**:
- `isHeld()` - On hold?
- `isMuted()` - Muted?
- `isSpeaker()` - Using speaker?
- `isTerminated()` - Call ended?

**Media Information**:
- `getRemoteOfferer()` - Who sent SDP offer?
- `getRemoteAudioCount()` - Remote audio streams offered
- `getRemoteVideoCount()` - Remote video streams offered
- `getAudioCount()` - Active audio streams
- `getVideoCount()` - Active video streams
- `getMedia()` - Current media state
- `getProvisionalMedia()` - Provisional media state

**Status Codes** (SIP):
- `getLastStatusCode()` - Last SIP status code (100-699)
- `getLastReason()` - Reason phrase

**Key Patterns**:
1. **Time Compensation**: Duration calculated with offset from construction time
2. **URI Parsing**: Regex extraction of name/number from SIP URIs
3. **Immutable Snapshot**: Data from native, no setters
4. **State Machine**: SIP INVITE states (NULL, CALLING, INCOMING, EARLY, CONNECTING, CONFIRMED, DISCONNECTED)

**Operations** (all in Endpoint, not Call):
- makeCall, answerCall, hangupCall, holdCall, unholdCall
- muteCall, unMuteCall, useSpeaker, useEarpiece
- xferCall, xferReplacesCall, redirectCall
- dtmfCall

## Flow Recommendation

- **Type**: TDD (correctness-critical state machine)
- **Confidence**: high
- **Rationale**: Call state machine is correctness-critical, duration tracking must be accurate

## Bubble Up

- Call is rich state holder with 30+ getters
- Time compensation for accurate duration tracking
- SIP URI parsing with regex
- Immutable snapshot pattern
- State machine with 7 states

---

*Phase: EXPLORING | Depth: 1 | Parent: root*
