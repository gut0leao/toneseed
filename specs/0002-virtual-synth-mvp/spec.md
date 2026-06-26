# Spec 0002: Virtual Synth MVP

## Status

Draft

## Context

ToneSeed should transform characteristics from a reference audio file into patches for real or virtual synthesizers. The microKORG Mk1 remains an important hardware target, but it should not be the first technical risk for the MVP.

The first validatable prototype should reduce dependencies on physical hardware, MIDI cabling, audio interfaces, SysEx write safety, and external audio capture. A controllable virtual synthesizer makes the core loop easier to run, test, repeat, and review in a development environment.

## Goal

Build the first ToneSeed MVP around a controllable virtual synthesizer target.

The MVP should prove that ToneSeed can:

- analyze a target audio file
- create a simple Tone IR
- map the Tone IR into a normalized patch
- translate the normalized patch through a synth-specific driver
- control a virtual synth through MIDI and/or automation
- render or capture the generated audio
- compare generated audio features against the target audio features

The goal is to generate patches inspired by the reference audio, not exact copies of the original sound.

## Initial Target Suggestion

The suggested first softsynth target is **Surge XT**.

Reasons:

- free and open source
- cross-platform
- broad synthesis feature set
- suitable as a generalist synth target
- realistic enough to exercise oscillator, filter, envelope, LFO, and effect mappings

The implementation should still avoid hard-coding the architecture around Surge XT. Surge XT is the first target used to validate the driver abstraction.

## Scope

- Define a common `SynthDriver` contract.
- Define a normalized patch format that is independent from any one synth.
- Implement or specify an initial virtual synth driver.
- Map a simple Tone IR into a normalized patch.
- Send note events to the virtual synth.
- Send parameter changes through the most practical mechanism available for the chosen target, such as MIDI CC, host automation, preset state, or a command-line/rendering bridge.
- Render or capture the generated audio in a repeatable way.
- Compare target and generated audio with simple feature metrics.
- Save run metadata, generated patches, captures/renders, and comparison scores.
- Keep the CLI-oriented workflow small and reviewable.

## Out Of Scope

- Physical hardware support in the first MVP.
- microKORG SysEx write support.
- Audio interface capture from an external synth.
- GUI application.
- Plugin authoring with VST/AU/AAX as a deliverable.
- Training neural models.
- Exact timbre cloning.
- Full preset compatibility with any synth.
- Support for many synths at once.

## Proposed Pipeline

```text
target audio
      ↓
analysis
      ↓
Tone IR
      ↓
Tone IR -> normalized patch mapping
      ↓
normalized patch
      ↓
SynthDriver implementation
      ↓
softsynth target
      ↓
MIDI / automation
      ↓
render / capture
      ↓
comparison
      ↓
run metadata
```

## Driver Abstraction

ToneSeed should introduce a common synth driver boundary before adding hardware-specific behavior.

Example contract:

```text
SynthDriver
  - id
  - name
  - list_parameters()
  - validate_patch(normalized_patch)
  - apply_patch(normalized_patch)
  - play_note(note, velocity, duration)
  - render_or_capture(note, duration, output_path)
```

Drivers should be responsible for translating normalized ToneSeed patch values into target-specific controls. They should not analyze audio, compute timbre similarity, or own the optimization loop.

The same boundary should later support:

- Surge XT and other VST/softsynth targets
- microKORG Mk1
- other hardware synthesizers
- generic MIDI targets

## Suggested Data Flow

```text
Audio Analysis
  -> Tone IR
  -> Normalized Patch
  -> SynthDriver
  -> Target-specific messages or automation
  -> Rendered/captured audio
  -> Comparison score
```

The normalized patch should use stable, synth-agnostic concepts where possible:

- oscillator waveform or oscillator character
- oscillator mix
- noise amount
- filter cutoff
- filter resonance
- amp envelope
- filter envelope
- modulation depth
- modulation rate
- effect amount

## CLI Sketch

The exact commands can evolve, but the first workflow should resemble:

```text
toneseed analyze examples/audio/target.wav
toneseed synths
toneseed play-note --synth surge-xt --note C3 --duration 1
toneseed grow examples/audio/target.wav --synth surge-xt
```

## Success Criteria

- [ ] The repository documents the Virtual Synth MVP as the first validatable prototype.
- [ ] A `SynthDriver` contract is defined before target-specific drivers grow.
- [ ] Tone IR, normalized patch, and synth driver responsibilities are separate.
- [ ] A target audio file can be analyzed into a serialized Tone IR.
- [ ] A normalized patch can be produced from the Tone IR.
- [ ] A virtual synth target can receive a note event.
- [ ] A virtual synth target can receive at least one meaningful parameter change.
- [ ] Generated audio can be rendered or captured reproducibly.
- [ ] Target and generated audio can be compared with simple feature metrics.
- [ ] Run metadata records the target audio, synth target, normalized patch, generated audio path, and comparison score.

## Risks

- Surge XT control surfaces may require host-specific automation or a plugin host bridge.
- MIDI CC mappings may not cover enough synthesis parameters for meaningful patch generation.
- Offline rendering may be more complex than real-time capture depending on the chosen host.
- Cross-platform behavior may differ across Linux, macOS, and Windows.
- Simple feature comparison may not align with human perception.
- A virtual-first implementation could accidentally assume capabilities that hardware targets do not have.

## Open Decisions

- Which runtime should host/control the virtual synth during the MVP?
- Should the first render path be offline rendering or real-time capture?
- Which Surge XT parameter set should be mapped first?
- Should normalized patch values be limited to `0.0` to `1.0`, typed enums, or both?
- What is the smallest useful comparison score for the first optimization loop?
- Where should run artifacts live: `runs/`, `experiments/runs/`, or another path?
- How should the CLI represent missing synth dependencies?

## Relationship To Spec 0001

[Spec 0001: microKORG Mk1 Hardware Target Spec](../0001-microkorg-mk1-mvp/spec.md) is preserved as a hardware target specification.

Spec 0002 supersedes Spec 0001 only as the first MVP execution order. It does not invalidate the microKORG work. Instead, the Virtual Synth MVP should establish the common architecture that the microKORG driver will later reuse:

```text
analysis -> Tone IR -> normalized patch -> SynthDriver -> target-specific implementation
```

After the virtual loop is validated, the microKORG can become a hardware MVP target using the same driver boundary, with additional hardware-specific concerns such as MIDI ports, SysEx backup, safe patch slots, audio interface capture, and latency calibration.
