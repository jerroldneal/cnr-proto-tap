# cnr-proto-tap

Enhanced Protocol Buffer WebSocket Tap v2 — intercepts all WebSocket connections on the page, decodes binary protobuf frames using the game's own `window.protobuf` library, and emits structured JSON events.

Consolidated from two previously divergent copies (cnr-ws-client-protobuf and cnr-ws-extension) into a single source of truth.

## Features

- **WebSocket proxy** — intercepts all non-localhost WebSocket connections
- **Dual-format decode** — length-prefixed (msgId) AND wrapper-based (topic string)
- **In-page event bus** — `on(topic, fn)` / `off(topic, fn)` with wildcard `*` support
- **Socket registry** — tracks all game WebSockets (`sockets` Map)
- **Action sender** — `sendAction(roomId, action, coin)` to send game actions
- **Ring buffers** — last 200 events + last 500 unknown frames
- **Relay broadcast** — streams decoded events to `ws://localhost:13030/ws`
- **Send-hook discovery** — catches sockets created before the proxy via `send()` override
- **Room tracking** — auto-tracks `roomId` from decoded messages

## Injection

Must be injected at `document_start` in `MAIN` world so the WebSocket proxy is installed before the game creates its connections.

## API

```javascript
// Event subscription
window.__ProtoTap.on('HoleCardsMsg', (evt) => console.log(evt));
window.__ProtoTap.on('*', (evt) => console.log(evt.topic, evt));
window.__ProtoTap.off('HoleCardsMsg', handler);

// Send action
window.__ProtoTap.sendAction(roomId, 5, 600); // RAISE 600

// State
window.__ProtoTap.roomId;        // last seen room ID
window.__ProtoTap.lastEvents;    // ring buffer of last 200 events
window.__ProtoTap.unknownFrames; // ring buffer of unknown msgId frames
window.__ProtoTap.sockets;       // Map of tracked WebSockets
window.__ProtoTap.stats();       // diagnostics

// Late-inject fallback
window.__ProtoTap.attachToSocket(existingWs);
```

## License

MIT
