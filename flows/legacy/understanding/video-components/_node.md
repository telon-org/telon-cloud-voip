# Understanding: Video Components

> Native video view components for local preview and remote party streams.

## Phase: EXPLORING

## Sources

- **src/PreviewVideoView.js** - Local camera preview (20 lines)
- **src/RemoteVideoView.js** - Remote party video (20 lines)

## Validated Understanding

**PreviewVideoView** - Local camera preview component.

```javascript
const PreviewVideoView = {
  name: 'PjSipPreviewVideoView',
  propTypes: {
    deviceId: PropTypes.number.isRequired,
    objectFit: PropTypes.oneOf(['contain', 'cover'])
  },
};
const View = requireNativeComponent('PjSipPreviewVideoView', null);
export default View;
```

**RemoteVideoView** - Remote party video component.

```javascript
const RemoteVideoView = {
  name: 'PjSipRemoteVideoView',
  propTypes: {
    windowId: PropTypes.string.isRequired,
    objectFit: PropTypes.oneOf(['contain', 'cover'])
  },
};
const View = requireNativeComponent('PjSipRemoteVideoView', null);
export default View;
```

**Characteristics**:
- Thin wrappers around `requireNativeComponent`
- PropTypes validation only
- No state, no behavior
- Native implementation handles all video rendering
- `objectFit` prop: 'contain' (letterbox) or 'cover' (crop)
- Preview uses `deviceId` (camera identifier)
- Remote uses `windowId` (native video window identifier)

## Flow Recommendation

- **Type**: VDD (Visual-Driven Development)
- **Confidence**: high
- **Rationale**: UI components, visual rendering, user experience primary

## Bubble Up

- Thin native component wrappers
- PropTypes validation only
- No state or behavior in JS
- objectFit prop for rendering mode

---

*Phase: EXPLORING | Depth: 1 | Parent: root*
