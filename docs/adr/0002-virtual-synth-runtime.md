# ADR 0002: Virtual Synth Runtime For The MVP

## Status

Accepted

## Context

The ToneSeed MVP will validate the core sound-generation loop with a virtual synth before any real hardware integration, especially before the microKORG Mk1.

The long-term goal still includes headless/offline execution, but requiring a custom headless renderer for the first prototype would add avoidable technical risk. The MVP needs a practical way to load a synth plugin, apply parameters, send MIDI, generate audio, capture that audio to WAV, and compare the result with a target file.

ToneSeed therefore needs two separate concepts:

- `SynthDriver`: translates a ToneSeed `NormalizedPatch` into synth-specific parameters.
- `SynthRuntime`: executes the synth, applies parameters, sends MIDI, captures or renders audio, and returns the generated WAV path.

## Decision

Introduce a separate `SynthRuntime` layer distinct from `SynthDriver`.

For the MVP, Carla is the first candidate external plugin host runtime.

Carla's role in the MVP is to:

- load the virtual synth plugin, initially Surge XT or another compatible plugin
- receive MIDI notes sent by ToneSeed
- receive parameters or automation through MIDI CC, OSC, or an equivalent supported mechanism
- play the virtual synth
- route the plugin audio output to an audio backend
- allow ToneSeed or a helper process to record that output to WAV

The MVP must not assume that Carla returns a WAV directly through an API. The initial strategy is to treat Carla as a plugin host and capture its audio output through routing, such as JACK, PipeWire, ALSA, or an equivalent mechanism available in the development environment.

The public runtime contract should be close to:

```python
render_patch(
    patch: NormalizedPatch,
    midi_sequence: MidiSequence,
    output_wav: Path,
) -> RenderResult
```

Conceptual flow:

```text
ToneSeed
  -> generates NormalizedPatch
  -> SynthDriver translates to synth parameters
  -> SynthRuntime loads/controls the host
  -> Carla loads the plugin
  -> ToneSeed sends MIDI and parameters
  -> plugin generates audio
  -> host audio output is captured
  -> ToneSeed saves render.wav
  -> ToneSeed compares render.wav with target.wav
```

## Consequences

Benefits:

- The MVP can use an existing plugin host instead of building plugin hosting first.
- `SynthDriver` stays focused on parameter translation.
- `SynthRuntime` owns execution, routing, capture, and host-specific lifecycle concerns.
- Carla can be replaced later by another runtime without changing Tone IR or normalized patch mapping.
- The architecture remains compatible with future headless/offline rendering.

Trade-offs:

- The first MVP depends on an external host and local audio routing.
- Capture may be environment-specific across JACK, PipeWire, ALSA, or platform equivalents.
- Tests may need a split between pure contract tests and manual/integration runtime validation.
- Real-time routed capture may be less deterministic than a future offline renderer.

## Alternatives Considered

### Surge XT Standalone

Surge XT standalone is useful for manual testing and quick sound checks, but it is fragile as an automation target. It does not provide a stable enough automation boundary for ToneSeed's first repeatable MVP loop.

### Virtual Synth Inside A DAW

A DAW is musically powerful and can host plugins, automation, routing, and rendering. It would also couple the MVP too tightly to a large interactive application and make repeatable CLI-driven validation harder.

### External Host Such As Carla

Carla reduces risk by acting as a controllable external plugin host. It can load plugin synths, receive MIDI and automation/control messages, and expose audio outputs that can be captured by the development environment.

This is the chosen initial approach.

### Custom Or Dedicated Headless/Offline Host

A headless/offline runtime remains the desired long-term direction because it can be more deterministic and easier to automate in CI-like workflows. It will not be required for the first MVP because implementing or integrating that layer upfront would increase the prototype risk.
