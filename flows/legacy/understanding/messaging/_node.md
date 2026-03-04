# Understanding: Messaging

> SIP instant messaging and typing notifications.

## Phase: EXPLORING

## Sources

- **src/Message.js** - Message data wrapper (100 lines)
- **src/Endpoint.js** - sendMessage, imTyping methods

## Validated Understanding

**Message Class** - Immutable data wrapper for SIP MESSAGE.

**Constructor Parsing**:
- Extracts name/number from From URI using regex
- Same pattern as Call.js

**Properties**:
- `getAccountId()` - Parent account ID
- `getContactUri()` - Sender's Contact URI
- `getFromUri()` - Sender URI
- `getFromName()` - Sender name (or null)
- `getFromNumber()` - Sender number
- `getToUri()` - Destination URI
- `getBody()` - Message body
- `getContentType()` - MIME type (e.g., "text/plain")

**Operations** (in Endpoint):
- `sendMessage(account, destination, msg)` - Send instant message
- `imTyping(account, destination, isTyping)` - Send typing indicator

**Pattern**:
- Pure data wrapper, no behavior
- Immutable snapshot from native
- No threading, no message history

## Flow Recommendation

- **Type**: SDD
- **Confidence**: medium
- **Rationale**: Simple data structure, internal service logic

## Bubble Up

- Message is simple immutable data wrapper
- URI parsing pattern (same as Call)
- No behavior, all operations in Endpoint
- Typing indicators supported

---

*Phase: EXPLORING | Depth: 1 | Parent: root*
