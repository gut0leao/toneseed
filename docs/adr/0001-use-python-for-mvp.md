# ADR 0001: Use Python For The MVP

## Status

Accepted

## Context

ToneSeed precisa analisar audio, enviar MIDI, capturar audio da interface, comparar features e iterar rapidamente sobre mapeamentos para o microKORG Mk1.

O projeto ainda esta em fase de pesquisa aplicada. A velocidade de experimentacao importa mais do que performance maxima.

## Decision

Usar Python como linguagem principal do MVP.

Bibliotecas candidatas:

- `librosa` para analise de audio
- `numpy` e `scipy` para processamento numerico
- `sounddevice` para audio I/O
- `soundfile` para leitura e escrita de WAV
- `mido` para mensagens MIDI
- `python-rtmidi` para MIDI em tempo real
- `pytest` para validacao automatizada

## Consequences

Vantagens:

- prototipagem rapida
- ecossistema forte para audio, MIR e otimizacao
- facil criacao de CLI
- boa integracao com notebooks e experimentos

Trade-offs:

- distribuicao de app desktop pode exigir trabalho adicional
- performance de baixo nivel pode exigir Rust, C++ ou extensoes no futuro
- dependencias de audio/MIDI podem variar entre sistemas operacionais

## Follow-up

Reavaliar a decisao se:

- o loop de otimizacao ficar limitado por performance
- houver necessidade de plugin VST/AU
- o projeto exigir baixa latencia em tempo real
