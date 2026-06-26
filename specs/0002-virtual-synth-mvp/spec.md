# Spec 0002: Virtual Synth MVP

## Status

Draft

## Context

ToneSeed should transform characteristics from a reference audio file into patches for real or virtual synthesizers. The microKORG Mk1 remains an important hardware target, but it should not be the first technical risk for the MVP.

The first validatable prototype should reduce dependencies on physical hardware, MIDI cabling, audio interfaces, SysEx write safety, and external audio capture. A controllable virtual synthesizer makes the core loop easier to run, test, repeat, and review in a development environment.

The MVP should use an external plugin host runtime before ToneSeed attempts a custom headless/offline renderer. The first runtime candidate is Carla.

## Goal

Build the first ToneSeed MVP around a controllable virtual synthesizer target.

The MVP should prove that ToneSeed can:

- analyze a target audio file
- create a simple Tone IR
- map the Tone IR into a normalized patch
- translate the normalized patch through a synth-specific driver
- execute the synth through a separate runtime layer
- control a virtual synth through MIDI and/or automation
- capture the generated audio to a WAV file
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

## Initial Runtime Suggestion

The suggested first runtime candidate is **Carla**.

Carla should be documented as an external plugin host used to reduce MVP risk. Its role is to load the virtual synth plugin, receive notes and parameter/control messages, play the synth, route audio to an available backend, and expose output that ToneSeed or a helper process can capture.

The MVP must not assume that Carla directly returns a WAV through an API. The initial render strategy is routed capture from the host output, using JACK, PipeWire, ALSA, or an equivalent mechanism available in the development environment.

The architecture must preserve the option to replace Carla later with another runtime, including a dedicated headless/offline runtime.

## Scope

- Define a common `SynthDriver` contract.
- Define a common `SynthRuntime` contract.
- Define a normalized patch format that is independent from any one synth.
- Implement or specify an initial virtual synth driver.
- Implement or specify an initial runtime using Carla as an external host candidate.
- Map a simple Tone IR into a normalized patch.
- Send note events to the virtual synth.
- Send parameter changes through the most practical mechanism available for the chosen target, such as MIDI CC, host automation, preset state, or a command-line/rendering bridge.
- Capture the host output to a WAV file in a repeatable way.
- Compare target and generated audio with simple feature metrics.
- Save run metadata, generated patches, captures/renders, and comparison scores.
- Keep the CLI-oriented workflow small and reviewable.

## Out Of Scope

- Physical hardware support in the first MVP.
- microKORG SysEx write support.
- Audio interface capture from an external synth.
- GUI application.
- Plugin authoring with VST/AU/AAX as a deliverable.
- Custom headless/offline plugin hosting as an MVP requirement.
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
target-specific parameters
      ↓
SynthRuntime implementation
      ↓
Carla external plugin host
      ↓
softsynth plugin
      ↓
MIDI / automation
      ↓
audio routing capture
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
  - translate_patch(normalized_patch)
```

Drivers should be responsible for translating normalized ToneSeed patch values into target-specific controls. They should not analyze audio, compute timbre similarity, own the optimization loop, host plugins, send MIDI, or capture audio.

The same boundary should later support:

- Surge XT and other VST/softsynth targets
- microKORG Mk1
- other hardware synthesizers
- generic MIDI targets

## Runtime Abstraction

ToneSeed should introduce a separate runtime boundary for synth execution.

Example contract:

```python
render_patch(
    patch: NormalizedPatch,
    midi_sequence: MidiSequence,
    output_wav: Path,
) -> RenderResult
```

`SynthRuntime` is responsible for:

- loading or connecting to the synth execution environment
- applying translated synth parameters
- sending MIDI notes or MIDI sequences
- coordinating automation/control delivery
- routing synth audio output
- capturing or rendering audio to `output_wav`
- returning a `RenderResult` with the generated WAV path and runtime metadata

For the MVP, the first candidate runtime is Carla. Carla should load Surge XT or another compatible plugin, receive MIDI and parameter/control messages from ToneSeed, route plugin audio to an available backend, and allow ToneSeed or a helper process to capture that output as WAV.

The runtime boundary must make Carla replaceable. Future runtimes may include another external host, a DAW bridge, a dedicated offline renderer, or a custom headless host.

## Suggested Data Flow

```text
Audio Analysis
  -> Tone IR
  -> Normalized Patch
  -> SynthDriver
  -> Target-specific parameters
  -> SynthRuntime
  -> Carla external plugin host
  -> Plugin audio output
  -> Routed audio capture
  -> render.wav
  -> Comparison score
```

Expanded MVP flow:

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
- [ ] A `SynthRuntime` contract is defined separately from `SynthDriver`.
- [ ] Tone IR, normalized patch, synth driver, and synth runtime responsibilities are separate.
- [ ] A target audio file can be analyzed into a serialized Tone IR.
- [ ] A normalized patch can be produced from the Tone IR.
- [ ] A virtual synth target can receive a note event.
- [ ] A virtual synth target can receive at least one meaningful parameter change.
- [ ] Generated audio can be captured from the host output to a WAV file reproducibly enough for MVP validation.
- [ ] Target and generated audio can be compared with simple feature metrics.
- [ ] Run metadata records the target audio, synth target, normalized patch, generated audio path, and comparison score.

## Risks

- Surge XT control surfaces may require host-specific automation or a plugin host bridge.
- Carla control and routing may differ across JACK, PipeWire, ALSA, and other platform audio backends.
- MIDI CC mappings may not cover enough synthesis parameters for meaningful patch generation.
- Real-time routed capture may be less deterministic than a later offline renderer.
- Cross-platform behavior may differ across Linux, macOS, and Windows.
- Simple feature comparison may not align with human perception.
- A virtual-first implementation could accidentally assume capabilities that hardware targets do not have.

## Open Decisions

- Which Carla control mechanism should the MVP try first: MIDI CC, OSC, host automation, or another supported path?
- Which audio routing backend should be the documented default for the development environment?
- Which Surge XT parameter set should be mapped first?
- Should normalized patch values be limited to `0.0` to `1.0`, typed enums, or both?
- What is the smallest useful comparison score for the first optimization loop?
- Where should run artifacts live: `runs/`, `experiments/runs/`, or another path?
- How should the CLI represent missing synth dependencies?

## Relationship To Spec 0001

[Spec 0001: microKORG Mk1 Hardware Target Spec](../0001-microkorg-mk1-mvp/spec.md) is preserved as a hardware target specification.

Spec 0002 supersedes Spec 0001 only as the first MVP execution order. It does not invalidate the microKORG work. Instead, the Virtual Synth MVP should establish the common architecture that the microKORG driver will later reuse:

```text
analysis -> Tone IR -> normalized patch -> SynthDriver -> SynthRuntime -> target-specific implementation
```

After the virtual loop is validated, the microKORG can become a hardware MVP target using the same driver boundary, with additional hardware-specific concerns such as MIDI ports, SysEx backup, safe patch slots, audio interface capture, and latency calibration.
