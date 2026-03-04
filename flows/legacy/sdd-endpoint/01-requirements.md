# Requirements: SIP Endpoint Core

> Spec-Driven Development flow for react-native-sip2 Endpoint module.

**Status**: DRAFT  
**Generated**: 2026-03-04  
**Source**: Legacy analysis of src/Endpoint.js

---

## Overview

The Endpoint module is the central coordinator for all SIP functionality in react-native-sip2. It provides a Promise-based API for SIP operations and manages event routing between native PjSIP module and application code.

---

## Functional Requirements

### FR-1: Module Initialization

**Description**: Application must initialize the PjSIP native module before any SIP operations.

**Requirements**:
- FR-1.1: Endpoint SHALL provide a `start(configuration)` method
- FR-1.2: `start()` SHALL return a Promise
- FR-1.3: `start()` SHALL resolve with accounts array, calls array, and extra configuration data
- FR-1.4: `start()` SHALL reject with error data if initialization fails
- FR-1.5: No SIP operations SHALL be called before `start()` completes

**Source**: Endpoint.start()

---

### FR-2: Account Management

**Description**: Endpoint SHALL manage SIP account lifecycle including creation, registration, and deletion.

**Requirements**:
- FR-2.1: Endpoint SHALL provide `createAccount(configuration)` method
- FR-2.2: `createAccount()` SHALL return a Promise resolving to Account instance
- FR-2.3: Endpoint SHALL provide `registerAccount(account, renew)` method
- FR-2.4: `registerAccount()` SHALL support renewal (true) and unregistration (false)
- FR-2.5: Endpoint SHALL provide `deleteAccount(account)` method
- FR-2.6: `deleteAccount()` SHALL unregister account from SIP server before deletion
- FR-2.7: Endpoint SHALL throw error for `replaceAccount()` (not implemented)

**Source**: Endpoint.createAccount(), registerAccount(), deleteAccount()

---

### FR-3: Call Operations

**Description**: Endpoint SHALL provide comprehensive call control operations.

**Requirements**:
- FR-3.1: Endpoint SHALL provide `makeCall(account, destination, callSettings, msgData)`
- FR-3.2: Endpoint SHALL provide `answerCall(call)`
- FR-3.3: Endpoint SHALL provide `hangupCall(call)`
- FR-3.4: Endpoint SHALL provide `declineCall(call)` (sends 603 response)
- FR-3.5: Endpoint SHALL provide `holdCall(call)` and `unholdCall(call)`
- FR-3.6: Endpoint SHALL provide `muteCall(call)` and `unMuteCall(call)`
- FR-3.7: Endpoint SHALL provide `useSpeaker(call)` and `useEarpiece(call)`
- FR-3.8: Endpoint SHALL provide `xferCall(account, call, destination)` (blind transfer)
- FR-3.9: Endpoint SHALL provide `xferReplacesCall(call, destCall)` (attended transfer)
- FR-3.10: Endpoint SHALL provide `redirectCall(account, call, destination)` (call forwarding)
- FR-3.11: Endpoint SHALL provide `dtmfCall(call, digits)` (DTMF tones)

**Source**: Endpoint call operation methods

---

### FR-4: Messaging

**Description**: Endpoint SHALL support SIP instant messaging.

**Requirements**:
- FR-4.1: Endpoint SHALL provide `sendMessage(account, destination, msg)`
- FR-4.2: Endpoint SHALL provide `imTyping(account, destination, isTyping)`
- FR-4.3: `sendMessage()` SHALL return Promise resolving on success
- FR-4.4: `imTyping()` SHALL send typing indicator to remote party

**Source**: Endpoint.sendMessage(), imTyping()

---

### FR-5: Event Emission

**Description**: Endpoint SHALL emit events for state changes and incoming communications.

**Requirements**:
- FR-5.1: Endpoint SHALL extend Node.js EventEmitter
- FR-5.2: Endpoint SHALL emit `registration_changed` with Account instance
- FR-5.3: Endpoint SHALL emit `call_received` with Call instance
- FR-5.4: Endpoint SHALL emit `call_changed` with Call instance
- FR-5.5: Endpoint SHALL emit `call_terminated` with Call instance
- FR-5.6: Endpoint SHALL emit `call_screen_locked` with boolean
- FR-5.7: Endpoint SHALL emit `message_received` with Message instance
- FR-5.8: Endpoint SHALL emit `connectivity_changed` with boolean or Account

**Source**: Endpoint event handlers

---

### FR-6: Configuration

**Description**: Endpoint SHALL provide runtime configuration methods.

**Requirements**:
- FR-6.1: Endpoint SHALL provide `updateStunServers(accountId, stunServerList)`
- FR-6.2: Endpoint SHALL provide `changeNetworkConfiguration(configuration)`
- FR-6.3: Endpoint SHALL provide `changeServiceConfiguration(configuration)`
- FR-6.4: Endpoint SHALL provide `changeCodecSettings(codecSettings)`
- FR-6.5: Endpoint SHALL provide `changeOrientation(orientation)` with validation
- FR-6.6: `changeOrientation()` SHALL validate against allowed orientations

**Source**: Endpoint configuration methods

---

### FR-7: Audio Session Management

**Description**: Endpoint SHALL manage iOS audio session activation.

**Requirements**:
- FR-7.1: Endpoint SHALL provide `activateAudioSession()`
- FR-7.2: Endpoint SHALL provide `deactivateAudioSession()`
- FR-7.3: Methods SHALL return Promises

**Source**: Endpoint audio session methods

---

### FR-8: SIP URI Normalization

**Description**: Endpoint SHALL automatically normalize destination URIs.

**Requirements**:
- FR-8.1: Endpoint SHALL convert bare numbers to SIP URI format
- FR-8.2: Endpoint SHALL use account's regServer or domain as realm
- FR-8.3: Endpoint SHALL preserve already-formatted SIP URIs

**Source**: Endpoint._normalize()

---

## Non-Functional Requirements

### NFR-1: Performance

- NFR-1.1: All native operations SHALL complete within 5 seconds under normal conditions
- NFR-1.2: Event emission SHALL be synchronous (EventEmitter pattern)

### NFR-2: Reliability

- NFR-2.1: Promise rejections SHALL occur for all native operation failures
- NFR-2.2: No retry logic SHALL be implemented (application responsibility)

### NFR-3: Maintainability

- NFR-3.1: All public methods SHALL be documented with JSDoc
- NFR-3.2: Code SHALL follow ESLint configuration (existing .eslintrc)

### NFR-4: Compatibility

- NFR-4.1: SHALL support React Native >=0.40.0 (peerDependencies)
- NFR-4.2: SHALL support iOS and Android platforms

---

## Constraints

- C-1: All operations depend on NativeModules.PjSipModule availability
- C-2: replaceAccount() is not implemented
- C-3: No error classification (raw data rejections)
- C-4: Event listeners are never removed (memory leak risk)

---

## Dependencies

- React Native (NativeModules, DeviceEventEmitter)
- Node.js events.EventEmitter
- Account, Call, Message data classes

---

*Generated by /legacy analysis*
