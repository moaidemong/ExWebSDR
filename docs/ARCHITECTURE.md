# ExWebSDR Architecture (MVP)

## Goal
- Build a real-time shortwave waterfall visualization using RTL-SDR v4.
- Provide stable browser rendering with low-latency updates and operator-friendly metadata.

## Current MVP Decisions
- Hardware access: Python RTL-SDR API direct control, single producer process per SDR stream.
- Processing: `core_producer.py` computes FFT + dB scaling and optional AM hover audio extraction.
- Backend bus: shared memory, latest-only policy.
- Browser transport: WebSocket binary for waterfall frames + JSON for metadata/control streams.
- Renderer: 2D canvas waterfall renderer optimized for clarity and low overhead.

## Pipeline
1. `core_producer.py` reads IQ from `sim` or `rtlsdr` source.
2. Producer applies Nuttall window, computes 8192-bin FFT, scales to dB, and encodes to `uint8` row.
3. Producer writes the latest waterfall row to shared memory.
4. `ws_server.py` reads the latest row and pushes it to connected browser clients.
5. Browser (`client/index.html`) renders each row into a scrolling waterfall.
6. Optional path: browser hover frequency -> gateway control file -> producer AM extract -> gateway JSON waveform -> browser overlay.

## Data Contract
### Waterfall shared-memory payload
- Header layout: `<I d H` (little-endian)
  - `uint32 seq`
  - `float64 t_capture_epoch_sec` (Unix epoch seconds)
  - `uint16 bins`
- Row payload: `uint8[bins]`
- Fixed runtime bin count: `8192`

### AM hover audio shared-memory payload
- Header layout: `<I d I H` (little-endian)
  - `uint32 seq`
  - `float64 t_capture_epoch_sec`
  - `uint32 tuned_freq_hz`
  - `uint16 sample_count`
- Payload: `int16[sample_count]` PCM
- Current producer settings:
  - Audio sample rate: `8000 Hz`
  - Chunk size: `160 samples`

### Scaling and FFT defaults
- dB clamp range: `[-110 dB, -20 dB]`
- Encoding: `u8 = round((clamp(db, -110, -20) + 110) * 255 / 90)`
- FFT bins: `8192`
- Hop size: `4096` (50% overlap)
- Window: 4-term Nuttall

## Runtime Topology
- Host: Windows 10/11
- Linux runtime: WSL2 Ubuntu 22.04
- Components:
  - `core_producer.py`
  - shared memory segments (`exwebsdr_*`)
  - `ws_server.py`
  - browser client
- Scheduler mode (`scheduler.py`) supports day-part band switching with A/B producer handover.

## Performance Targets (MVP)
- End-to-end latency: `<= 120 ms` p95.
- Browser render rate: `>= 20 FPS` sustained.
- Frame drop rate: `< 5%` over 5 minutes.
- Recovery time after disconnect: `<= 3 s`.

## Failure and Backpressure Policy
- Shared memory keeps latest frame only (no deep queue).
- Gateway may skip stale rows and publish newest frame.
- On producer stop/failure, shared memory is recreated on restart and gateway re-attaches.

## Out of Scope (MVP)
- Distributed bus or multi-node deployment
- Long-term storage/replay pipeline
- Advanced 3D renderer pipeline

## Related Docs
- `docs/PROTOCOL.md`: gateway-client frame/message protocol.
- `docs/OPERATIONS.md`: scheduler and service operation.
