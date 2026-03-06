# ExWebSDR WebSocket Protocol (MVP)

## Scope
- Defines runtime messages between `ws_server.py` and browser clients.
- Covers waterfall binary frames and JSON side channels (`meta`, `am_wave`, hover control).

## Transport
- Protocol: WebSocket
- Waterfall: binary frames
- Metadata and AM waveform: JSON text frames
- Delivery policy: latest-oriented, stale rows may be skipped

## Binary Waterfall Frame
Little-endian layout (`HEADER_FORMAT = "<I d H"`):
1. `uint32 seq`
2. `float64 t_capture_epoch_sec`
3. `uint16 bins`
4. `uint8[bins] row`

### Current fixed values
- `bins = 8192`
- Total frame length: `14 + 8192 = 8206` bytes
- dB clamp range encoded in producer: `[-110, -20]`

### Client validation rules
- Reject if payload length `< 14`
- Reject if `bins <= 0`
- Reject if payload length `!= 14 + bins`
- Reject if `bins != 8192` (current strict mode in gateway/client)

## JSON Messages From Gateway
### `meta`
```json
{
  "type": "meta",
  "band_name": "Night SW",
  "center_freq_hz": 6850000,
  "sample_rate_hz": 2400000,
  "next_change_epoch_sec": 1770000000.0
}
```

### `am_wave`
```json
{
  "type": "am_wave",
  "freq_hz": 7000000,
  "sample_rate_hz": 8000,
  "samples": [0.01, -0.03, 0.04]
}
```

## JSON Message From Client
### `hover`
```json
{
  "type": "hover",
  "mode": "am",
  "freq_hz": 7000000,
  "strength": 0.42
}
```

Gateway writes this intent to `runtime/hover_control.json` (rate-limited), and producer uses it to generate AM waveform chunks.

## Sequence and Timing
- `seq` increases by 1 per producer frame/chunk.
- `t_capture_epoch_sec` is Unix epoch seconds from producer capture loop.
- Frame-drop estimation can use sequence gaps.
- Approximate latency estimate:
  - `latency_ms = (Date.now() / 1000 - t_capture_epoch_sec) * 1000`

## Error Handling
- If producer/shared memory is unavailable, gateway sends no synthetic frame.
- If active shared-memory name changes (scheduler handover), gateway re-attaches to the new segment.
- AM waveform stream is optional; client should continue waterfall render if AM data is absent.
