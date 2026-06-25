# Spec 0001: microKORG Mk1 MVP

## Status

Ready

## Context

ToneSeed deve comecar com um alvo concreto: Korg microKORG Mk1 conectado a uma interface M-Audio Fast Track C400.

O projeto precisa provar o ciclo minimo:

```text
audio alvo -> analise -> Tone IR -> patch -> microKORG -> captura -> comparacao
```

Este MVP deve ser pequeno, testavel e orientado por CLI.

## Goal

Criar a primeira versao funcional do ToneSeed capaz de:

- analisar um arquivo WAV
- representar o timbre em uma Tone IR simples
- tocar notas MIDI no microKORG Mk1
- capturar audio gerado pelo microKORG
- preparar o caminho para envio de parametros via MIDI/SysEx

## Non-goals

- Criar interface grafica.
- Suportar outros sintetizadores.
- Treinar modelo neural.
- Reproduzir o audio alvo com fidelidade literal.
- Implementar plugin VST/AU.
- Fazer orquestracao com multiplos agentes.

## User Workflow

```text
Usuario conecta microKORG e Fast Track C400
  ↓
Usuario lista dispositivos
  ↓
Usuario testa nota MIDI
  ↓
Usuario analisa WAV alvo
  ↓
ToneSeed gera Tone IR
  ↓
ToneSeed gera patch candidato
  ↓
ToneSeed toca e captura o microKORG
  ↓
ToneSeed compara features
```

## CLI Contract

Comandos planejados:

```text
toneseed devices
toneseed analyze <audio.wav>
toneseed play-note --note C3 --duration 1
toneseed capture --note C3 --duration 2 --output captures/c3.wav
toneseed grow <audio.wav> --synth microkorg-mk1
```

## Functional Requirements

- O sistema deve listar portas MIDI disponiveis.
- O sistema deve listar dispositivos de audio disponiveis.
- O sistema deve enviar `note_on` e `note_off` para uma porta MIDI selecionada.
- O sistema deve ler arquivos WAV.
- O sistema deve extrair features basicas do audio.
- O sistema deve serializar uma Tone IR em JSON.
- O sistema deve capturar audio de uma entrada selecionada.
- O sistema deve salvar capturas em WAV.
- O sistema deve comparar features do alvo e da captura.
- O sistema deve salvar metadados de cada tentativa.

## Technical Requirements

- Linguagem principal: Python.
- Interface inicial: CLI.
- Audio analysis deve ficar separado de drivers de sintetizador.
- Driver microKORG Mk1 deve ficar separado de captura de audio.
- Tone IR deve ser independente de hardware.
- Parametros de patch devem ser serializaveis em JSON.
- SysEx deve comecar por leitura/backup antes de escrita destrutiva.

## Suggested Modules

```text
toneseed/
    cli.py
    analysis/
        features.py
    ir/
        tone.py
    midi/
        devices.py
        notes.py
    audio/
        devices.py
        capture.py
    drivers/
        microkorg_mk1.py
    optimization/
        compare.py
```

## Data Contracts

### Tone IR

```json
{
  "pitch": {
    "estimated_hz": 110.0,
    "stability": 0.82
  },
  "brightness": 0.74,
  "noise": 0.18,
  "harmonicity": 0.67,
  "envelope": {
    "attack_ms": 35,
    "decay_ms": 420,
    "sustain": 0.58,
    "release_ms": 280
  },
  "motion": {
    "spectral_change": 0.21,
    "amplitude_modulation": 0.12
  }
}
```

### Attempt Metadata

```json
{
  "target_audio": "examples/audio/target.wav",
  "synth": "microkorg-mk1",
  "note": "C3",
  "capture": "captures/attempt-001.wav",
  "patch": "runs/attempt-001.patch.json",
  "score": 0.64
}
```

## Acceptance Criteria

- [ ] `toneseed devices` lista portas MIDI e dispositivos de audio.
- [ ] `toneseed play-note --note C3 --duration 1` toca uma nota no microKORG Mk1.
- [ ] `toneseed analyze examples/audio/target.wav` gera uma Tone IR em JSON.
- [ ] `toneseed capture --note C3 --duration 2 --output captures/c3.wav` salva uma captura WAV.
- [ ] O projeto tem testes para conversao de notas MIDI.
- [ ] O projeto tem testes para serializacao da Tone IR.
- [ ] O projeto separa analise, MIDI, audio, driver e otimizacao em modulos distintos.
- [ ] A documentacao explica como conectar o hardware.

## Validation Plan

Validacao sem hardware:

```text
pytest
python -m toneseed --help
toneseed analyze examples/audio/target.wav
```

Validacao com hardware:

```text
toneseed devices
toneseed play-note --note C3 --duration 1
toneseed capture --note C3 --duration 2 --output captures/c3.wav
```

## Risks

- SysEx do microKORG Mk1 pode exigir investigacao por dumps reais.
- Comparacao de timbre por features simples pode nao refletir percepcao humana.
- Latencia de audio/MIDI pode afetar a captura se nao houver calibracao.
- Busca de parametros pode ser lenta se o espaco de patch for grande.

## Open Questions

- Qual canal MIDI sera usado como padrao?
- Qual slot/banco do microKORG sera reservado para testes?
- A primeira captura sera sempre mono ou havera suporte stereo desde o inicio?
- O projeto deve salvar runs em `runs/` ou `experiments/runs/`?
