# Visual Requirements: Video Components

> Visual-Driven Development flow for PreviewVideoView and RemoteVideoView.

**Status**: DRAFT  
**Generated**: 2026-03-04  
**Source**: Legacy analysis of src/PreviewVideoView.js, src/RemoteVideoView.js

---

## Component Overview

**Components**:
- **PreviewVideoView**: Local camera preview
- **RemoteVideoView**: Remote party video stream

**Purpose**: Display video streams in React Native application with native performance.

---

## Visual Requirements

### VR-1: PreviewVideoView Rendering

**Description**: Local camera preview SHALL display real-time video from device camera.

**Requirements**:
- VR-1.1: PreviewVideoView SHALL render camera feed with minimal latency (<100ms)
- VR-1.2: PreviewVideoView SHALL support `contain` fit mode (letterbox)
- VR-1.3: PreviewVideoView SHALL support `cover` fit mode (crop to fill)
- VR-1.4: PreviewVideoView SHALL require `deviceId` prop (camera identifier)
- VR-1.5: PreviewVideoView SHALL update when `deviceId` changes

**Visual Specification**:
```
┌─────────────────────────┐
│  ┌───────────────────┐  │
│  │                   │  │
│  │   Camera Feed     │  │  <- Contain: black bars if aspect ratio mismatch
│  │                   │  │
│  └───────────────────┘  │
│                         │
└─────────────────────────┘
```

---

### VR-2: RemoteVideoView Rendering

**Description**: Remote party video SHALL display incoming video stream.

**Requirements**:
- VR-2.1: RemoteVideoView SHALL render remote video stream
- VR-2.2: RemoteVideoView SHALL support `contain` fit mode
- VR-2.3: RemoteVideoView SHALL support `cover` fit mode
- VR-2.4: RemoteVideoView SHALL require `windowId` prop (native window identifier)
- VR-2.5: RemoteVideoView SHALL update when video stream changes

**Visual Specification**:
```
┌─────────────────────────┐
│  ┌───────────────────┐  │
│  │                   │  │
│  │   Remote Video    │  │  <- Full screen or container
│  │                   │  │
│  └───────────────────┘  │
│                         │
└─────────────────────────┘
```

---

### VR-3: Aspect Ratio Handling

**Description**: Components SHALL handle various aspect ratios gracefully.

**Requirements**:
- VR-3.1: `contain` mode SHALL preserve aspect ratio with letterboxing
- VR-3.2: `cover` mode SHALL fill container with cropping
- VR-3.3: No distortion SHALL occur in either mode

**Visual Examples**:

**Contain (16:9 video in 4:3 container)**:
```
┌─────────────────────────┐
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  │  <- Black bars (letterbox)
│  ▓                   ▓  │
│  ▓    16:9 Video     ▓  │
│  ▓                   ▓  │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  │
└─────────────────────────┘
     4:3 Container
```

**Cover (16:9 video in 4:3 container)**:
```
┌─────────────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│ ▓▓▓  16:9 Video   ▓▓▓▓▓ │  <- Cropped edges
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
└─────────────────────────┘
     4:3 Container
```

---

### VR-4: Component Sizing

**Description**: Components SHALL adapt to container size.

**Requirements**:
- VR-4.1: Components SHALL fill parent container
- VR-4.2: Components SHALL respond to container resize
- VR-4.3: Components SHALL maintain native performance during resize

---

### VR-5: Multiple Video Streams

**Description**: Application MAY render multiple video components simultaneously.

**Requirements**:
- VR-5.1: Multiple PreviewVideoView components MAY coexist (multi-camera)
- VR-5.2: Multiple RemoteVideoView components MAY coexist (conference calls)
- VR-5.3: Each component SHALL render independent video stream
- VR-5.4: Performance SHALL remain acceptable with multiple streams

**Layout Example (Conference Call)**:
```
┌─────────────────────────────────────┐
│  ┌───────────┐  ┌───────────┐      │
│  │ Remote 1  │  │ Remote 2  │      │
│  └───────────┘  └───────────┘      │
│  ┌───────────┐  ┌───────────┐      │
│  │ Remote 3  │  │ Preview   │      │
│  └───────────┘  └───────────┘      │
└─────────────────────────────────────┘
```

---

## Props Specification

### PreviewVideoView Props

```javascript
PreviewVideoView.propTypes = {
  deviceId: PropTypes.number.isRequired,  // Camera identifier
  objectFit: PropTypes.oneOf(['contain', 'cover'])  // Rendering mode
};
```

**deviceId**:
- Type: `number`
- Required: Yes
- Description: Native camera device identifier
- Source: Obtained from native module or call media info

**objectFit**:
- Type: `'contain' | 'cover'`
- Required: No
- Default: `'contain'` (assumed)
- Description: Video scaling mode

---

### RemoteVideoView Props

```javascript
RemoteVideoView.propTypes = {
  windowId: PropTypes.string.isRequired,  // Native video window ID
  objectFit: PropTypes.oneOf(['contain', 'cover'])  // Rendering mode
};
```

**windowId**:
- Type: `string`
- Required: Yes
- Description: Native video renderer window identifier
- Source: Obtained from call media info

**objectFit**:
- Type: `'contain' | 'cover'`
- Required: No
- Default: `'contain'` (assumed)
- Description: Video scaling mode

---

## Usage Examples

### Basic Preview

```javascript
import {PreviewVideoView} from 'react-native-sip2';

function VideoPreview({deviceId}) {
  return (
    <PreviewVideoView
      deviceId={deviceId}
      objectFit="contain"
      style={{width: 200, height: 150}}
    />
  );
}
```

### Basic Remote Video

```javascript
import {RemoteVideoView} from 'react-native-sip2';

function RemoteVideo({windowId}) {
  return (
    <RemoteVideoView
      windowId={windowId}
      objectFit="cover"
      style={{flex: 1}}
    />
  );
}
```

### Conference Layout

```javascript
function ConferenceLayout({remotes, previewDeviceId}) {
  return (
    <View style={styles.container}>
      <View style={styles.grid}>
        {remotes.map((remote) => (
          <RemoteVideoView
            key={remote.windowId}
            windowId={remote.windowId}
            objectFit="contain"
            style={styles.videoTile}
          />
        ))}
      </View>
      <PreviewVideoView
        deviceId={previewDeviceId}
        objectFit="cover"
        style={styles.preview}
      />
    </View>
  );
}
```

---

## Performance Requirements

- **P-001**: Video rendering SHALL maintain >= 24 FPS
- **P-002**: Video latency SHALL be < 200ms end-to-end
- **P-003**: Component mount SHALL complete in < 100ms
- **P-004**: Prop updates SHALL apply in < 50ms
- **P-005**: Memory usage SHALL be stable (no leaks)

---

## Accessibility

- **A-001**: Video components SHOULD support screen readers
- **A-002**: Alternative text SHOULD describe video content
- **A-003**: Muted state SHOULD be indicated visually

---

## Platform Differences

### iOS

- Native component: `PjSipPreviewVideoView`, `PjSipRemoteVideoView`
- Video renderer: AVFoundation-based
- Orientation: Auto-rotates with device

### Android

- Native component: `PjSipPreviewVideoView`, `PjSipRemoteVideoView`
- Video renderer: SurfaceView/TextureView-based
- Orientation: May require manual handling

---

## Testing Guidelines

### Visual Tests

- Verify `contain` mode shows full video with letterboxing
- Verify `cover` mode fills container with cropping
- Verify aspect ratio preservation
- Verify multiple simultaneous renders

### Integration Tests

- Verify video appears when call established
- Verify video disappears when call terminates
- Verify deviceId/windowId changes update rendering

---

*Generated by /legacy analysis*
