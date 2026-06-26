# Architecture Decisions Before MVP Implementation

This document tracks the main ToneSeed architecture decisions that are already resolved and the decisions still open before implementation begins.

It is not an implementation task list. Its purpose is to make architectural alignment explicit for human contributors and AI agents.

## Resolved Decisions

### Virtual Synth First

Status: Resolved

The MVP will use a virtual synthesizer first.

Hardware MIDI targets, including the microKORG Mk1 and other physical synths, will be handled only after the main virtual synth architecture has been validated.

### Runtime Abstraction

Status: Resolved

The architecture clearly separates:

- `SynthDriver`
- `SynthRuntime`

`SynthDriver` translates a ToneSeed `NormalizedPatch` into synth-specific parameters.

`SynthRuntime` executes the synth, sends MIDI, applies parameters, captures or renders audio, and produces the generated audio artifact.

### MVP Runtime

Status: Partially Resolved

The first runtime will use Carla as an external plugin host.

The definitive audio capture mechanism remains open.

The architecture must continue to allow a future replacement with a headless/offline runtime and must not couple ToneSeed's core contracts to Carla.

## Open Decisions

### 1. Definitive Audio Capture Strategy

Status: Open

ToneSeed still needs to decide how the MVP will produce the generated WAV file.

Possible alternatives:

- JACK capture
- PipeWire capture
- ALSA capture
- offline rendering from the host
- another mechanism available in the development environment

Important note: the architecture must not depend on the chosen capture strategy. The public `SynthRuntime` contract remains stable.

### 2. Definitive SynthDriver Contract

Status: Open

ToneSeed still needs to define the exact `SynthDriver` contract:

- public interface
- required methods
- error handling
- return types
- responsibilities

The goal is to allow independent drivers for different synths without leaking runtime, capture, or analysis concerns into the driver layer.

### 3. NormalizedPatch Specification

Status: Open

ToneSeed still needs to define the `NormalizedPatch` specification:

- JSON structure
- parameter types
- enums
- normalized ranges
- versioning

This will be the main contract between analysis and synthesis.

### 4. Produced Artifact Organization

Status: Open

ToneSeed still needs to define the official artifact layout for:

- `target.wav`
- `render.wav`
- normalized patch
- Tone IR
- metrics
- metadata
- logs

The layout should support reproducible experiments and make it easy to compare runs.

### 5. Backlog Update

Status: Open

The backlog still needs to be reorganized around the current architecture:

- Virtual Synth MVP
- `SynthRuntime`
- Carla runtime
- dummy runtime
- Tone IR
- automatic comparison

microKORG-related items should depend on completion of the virtual synth MVP.

### 6. Dummy Runtime

Status: Open

ToneSeed should define a very simple runtime for validating the architecture without depending on real plugins.

The goal is to support incremental development and automated tests for the core pipeline.

### 7. Headless Strategy

Status: Open

After the Carla-based runtime stabilizes, ToneSeed should study alternatives for fully headless execution.

This is not part of the initial MVP, but it remains an important architectural goal.

## Summary

| Decision | Status | MVP blocker? |
| --- | --- | --- |
| Virtual Synth | Resolved | No |
| SynthRuntime | Resolved | No |
| Audio capture | Open | Yes |
| SynthDriver contract | Open | Yes |
| NormalizedPatch | Open | Yes |
| Artifacts layout | Open | No |
| Backlog update | Open | No |
| Dummy Runtime | Open | No |
| Headless Runtime | Open | No |
