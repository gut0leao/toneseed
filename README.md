# ToneSeed

> **Plant a sound. Grow a patch.**

ToneSeed is an AI-assisted sound design platform that transforms any audio recording into a synthesizer patch for real hardware.

Unlike samplers or SoundFonts, ToneSeed does **not** attempt to reproduce the original sound perfectly. Instead, it analyzes the sonic characteristics of a recording and generates a patch that captures its essence using the synthesis capabilities of a connected hardware synthesizer.

The result is not a clone.

The result is a **new sound inspired by the original**, expressed through the unique personality of the destination synthesizer.

## Vision

Every synthesizer has its own voice.

A Minimoog should sound like a Minimoog.

A Juno should sound like a Juno.

A Prophet should sound like a Prophet.

ToneSeed does not try to erase those differences.

Instead, it uses them as part of the creative process.

The original sound becomes a **seed** from which new sounds grow.

## Philosophy

Traditional approaches usually follow one of these paths:

- Sampling
- SoundFonts
- Wavetable extraction
- Physical modeling

ToneSeed proposes a different approach.

```text
Audio
      ↓
Spectral Analysis
      ↓
Feature Extraction
      ↓
AI Optimization
      ↓
Synthesizer Parameter Mapping
      ↓
Real Analog Hardware
      ↓
New Sound
```

The destination synthesizer is **not** merely a playback device.

It is an active participant in the creative process.

## Project Goals

The project aims to create a generic framework capable of translating sonic characteristics into synthesizer parameters.

Initially the focus will be hardware synthesizers connected through MIDI.

Future versions may support:

- Software synthesizers
- Eurorack modular systems
- CV/Gate interfaces
- Vintage synthesizers via SysEx
- FM synthesizers
- Wavetable synthesizers
- Digital workstations

## Core Concepts

### 1. Audio Analysis

Extract perceptually meaningful information from an audio recording, including:

- Spectral centroid
- Harmonic content
- Envelope
- Attack
- Decay
- Brightness
- Noise content
- Resonance estimation
- Pitch stability
- Temporal evolution

The analysis stage should remain independent from any synthesizer.

### 2. Timbre Representation

Create an intermediate representation (IR) describing the sound.

This representation should be hardware independent.

```text
Audio
      ↓
Tone IR
```

The IR becomes the bridge between analysis and synthesis.

### 3. Synth Drivers

Each supported synthesizer implements a driver capable of translating the Tone IR into its own parameter space.

Example:

```text
Tone IR
      │
      ├── Roland Juno
      ├── Minimoog
      ├── Prophet
      ├── Behringer Model D
      ├── DeepMind
      └── ...
```

Each synthesizer remains faithful to its own sonic identity.

### 4. Optimization Loop

Rather than estimating parameters only once, ToneSeed may iteratively improve the generated patch.

```text
Generate Patch

↓

Send MIDI / SysEx

↓

Play Note

↓

Capture Audio

↓

Compare with Target

↓

Adjust Parameters

↓

Repeat
```

Possible optimization techniques:

- Bayesian Optimization
- Genetic Algorithms
- Reinforcement Learning
- Gradient-free optimization
- Evolutionary Strategies

## User Experience

The intended workflow should be extremely simple.

```text
Select Audio

↓

Choose Synthesizer

↓

Grow Patch

↓

Play
```

Advanced users may adjust the creativity level.

```text
Faithfulness

Literal ───────────── Creative
```

The software should encourage exploration rather than exact reproduction.

## Design Principles

- Creativity before imitation
- Hardware-first
- Non-destructive workflow
- Synthesizer personality is preserved
- Extensible architecture
- Open research platform
- Spec-driven development

## Development Workflow

ToneSeed is being developed with an AI-assisted, spec-driven workflow.

The project keeps product context, technical specs, architectural decisions, and implementation backlog inside the repository so changes remain explicit, reviewable, and traceable.

Key documents:

- [AI Development Workflow](docs/AI_DEVELOPMENT_WORKFLOW.md)
- [microKORG Mk1 MVP Spec](specs/0001-microkorg-mk1-mvp/spec.md)
- [Hardware Setup](docs/HARDWARE_SETUP.md)
- [microKORG Mk1 Driver](docs/MICROKORG_MK1_DRIVER.md)
- [Backlog](BACKLOG.md)

## Possible Architecture

```text
tone-seed/

    analysis/
        Spectral analysis
        Feature extraction

    ir/
        Tone intermediate representation

    drivers/
        Juno
        Minimoog
        Prophet
        DeepMind
        Generic MIDI

    optimization/
        Genetic algorithms
        Bayesian optimization
        Reinforcement learning

    midi/
        MIDI
        SysEx
        Parameter transmission

    ui/
        Desktop application

    experiments/
        Research notebooks

    docs/
        Papers
        References
```

## Technology Candidates

Python

Libraries:

- librosa
- essentia
- numpy
- scipy
- mido
- python-rtmidi
- PyTorch
- ONNX Runtime

Future:

- Rust performance modules
- JUCE plugin
- C++ DSP core

## Research Topics

- Automatic Synthesizer Programming (ASP)
- Music Information Retrieval (MIR)
- Spectral Feature Extraction
- Audio Embeddings
- Differentiable DSP
- Neural Audio Synthesis
- Evolutionary Optimization
- Timbre Similarity Metrics

## Long-Term Vision

ToneSeed should become a universal timbre translation platform.

Possible future capabilities include:

- Audio -> Analog Synth
- Audio -> Modular Synth
- Audio -> FM Patch
- Patch -> Patch Translation
- Preset Migration Between Synthesizers
- AI-assisted Sound Design
- Automatic Patch Libraries

The long-term goal is not to replace synthesizers.

It is to help musicians discover sounds they would never have programmed themselves.

## Motto

> **Every sound is a seed.**
>
> **Grow something new.**
