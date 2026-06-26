# ToneSeed MVP: microKORG Mk1

> Architectural update: this document is preserved as the future hardware MVP / hardware target plan for the microKORG Mk1. The first validatable prototype should now be the Virtual Synth MVP described in `specs/0002-virtual-synth-mvp/spec.md`, so ToneSeed can validate analysis, Tone IR, normalized patch mapping, synth driver control, render/capture, and comparison before depending on physical hardware, SysEx, and external audio capture.

## Objetivo

Construir um MVP de hardware do ToneSeed tendo o Korg microKORG Mk1 como sintetizador alvo.

O MVP deve receber um arquivo de audio de referencia, extrair caracteristicas perceptuais, gerar uma representacao intermediaria simples e produzir um patch inspirado no audio original usando os recursos reais do microKORG Mk1.

O objetivo nao e reproduzir o som de forma literal. O objetivo e criar um novo patch que preserve aspectos relevantes do timbre de origem dentro da personalidade sonora do microKORG.

## Hardware Inicial

- Korg microKORG Mk1
- M-Audio Fast Track C400
- Cabo MIDI DIN 5 pinos: interface MIDI Out para microKORG MIDI In
- Cabo MIDI DIN 5 pinos: microKORG MIDI Out para interface MIDI In
- Cabo P10: microKORG Output L/Mono para entrada de linha da interface
- Cabo P10 adicional opcional: microKORG Output R para segunda entrada da interface
- Fones ou monitores para monitoramento

## Fluxo Do MVP

```text
Audio alvo
      ↓
Analise de features
      ↓
Tone IR simples
      ↓
Mapeamento para parametros do microKORG Mk1
      ↓
Envio MIDI / SysEx
      ↓
Execucao de nota MIDI
      ↓
Captura do audio gerado
      ↓
Comparacao com o audio alvo
      ↓
Ajuste iterativo
```

## Escopo Inicial

- Carregar um arquivo WAV mono ou stereo.
- Converter o audio para uma representacao mono para analise inicial.
- Extrair features basicas:
  - centroide espectral
  - rolloff espectral
  - conteudo harmonico
  - conteudo de ruido
  - envelope de amplitude
  - ataque
  - decaimento
  - estabilidade de pitch
  - RMS
- Criar uma Tone IR simples e serializavel.
- Mapear a Tone IR para parametros do microKORG Mk1.
- Enviar nota MIDI para disparar o synth.
- Capturar o audio resultante pela interface.
- Comparar audio alvo e audio capturado usando metricas simples.
- Registrar patches candidatos e resultados de comparacao.

## Fora Do Escopo Inicial

- Interface grafica.
- Suporte a multiplos sintetizadores.
- Plugin VST, AU ou AAX.
- Treinamento de modelos neurais.
- Reproducao perfeita do audio original.
- Controle de sintetizadores modulares, CV/Gate ou Eurorack.
- Suporte generico a qualquer equipamento MIDI.

## Stack Recomendada

- Python como linguagem principal.
- `librosa` para analise de audio.
- `numpy` e `scipy` para processamento numerico e DSP.
- `sounddevice` para captura e reproducao de audio.
- `soundfile` para leitura e escrita de WAV.
- `mido` para mensagens MIDI.
- `python-rtmidi` como backend MIDI em tempo real.
- `pydantic` ou `dataclasses` para representar a Tone IR.
- `pytest` para testes unitarios.

## Primeira Interface CLI

O MVP deve comecar como ferramenta de linha de comando.

```text
toneseed devices
toneseed analyze target.wav
toneseed play-note --note C3 --duration 2
toneseed capture --note C3 --duration 2 --output captures/test.wav
toneseed grow target.wav --synth microkorg-mk1
```

## Tone IR Inicial

Uma primeira versao da Tone IR pode conter apenas dados suficientes para gerar patches simples.

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

## Estrategia De Mapeamento Inicial

O primeiro mapeamento deve ser heuristico e explicito.

Exemplos:

- Brilho alto aumenta cutoff do filtro.
- Conteudo de ruido alto aumenta uso de noise oscillator ou formas de onda mais ruidosas.
- Ataque curto gera envelope rapido.
- Ataque longo gera envelope lento.
- Timbre mais harmonico favorece saw, square ou triangle.
- Timbre instavel aumenta vibrato ou mod wheel depth.
- Movimento temporal aumenta LFO ou modulation patch.

Essa camada deve ficar separada da analise de audio. A analise nao deve conhecer detalhes do microKORG.

## Loop De Otimizacao Inicial

A primeira versao do loop nao precisa usar machine learning.

```text
1. Gerar patch candidato.
2. Enviar patch para o microKORG.
3. Tocar nota MIDI fixa.
4. Capturar audio gerado.
5. Normalizar volume.
6. Extrair features do audio capturado.
7. Comparar features alvo e capturadas.
8. Ajustar parametros.
9. Repetir por N iteracoes.
```

Tecnicas iniciais:

- busca aleatoria com limites
- hill climbing
- simulated annealing simples
- `scipy.optimize` com algoritmo sem gradiente

## Milestones

1. Detectar portas MIDI e dispositivos de audio.
2. Tocar uma nota no microKORG via MIDI.
3. Capturar o audio do microKORG pela Fast Track C400.
4. Extrair features de um WAV alvo.
5. Criar a primeira Tone IR.
6. Implementar o primeiro mapeamento heuristico para o microKORG Mk1.
7. Enviar ou editar um patch via MIDI/SysEx.
8. Rodar uma primeira comparacao entre audio alvo e audio capturado.
9. Rodar o primeiro loop automatico de otimizacao.
10. Salvar o melhor patch encontrado.

## Riscos Tecnicos

- O formato SysEx do microKORG Mk1 precisa ser documentado e testado com cuidado.
- Alguns parametros podem nao responder em tempo real da mesma forma que respondem ao editar pelo painel.
- O envio de patches pode sobrescrever memoria do sintetizador se nao houver cuidado.
- Latencia de audio/MIDI precisa ser medida para alinhar captura e comparacao.
- Diferencas de ganho podem distorcer metricas de comparacao.
- O espaco de parametros do synth e grande para busca cega.

## Criterio De Sucesso Do MVP

O MVP sera considerado funcional quando for possivel executar:

```text
toneseed grow examples/audio/target.wav --synth microkorg-mk1
```

E obter:

- um patch gerado ou editado no microKORG Mk1
- uma captura WAV do audio produzido
- um arquivo JSON com os parametros usados
- uma pontuacao de similaridade baseada em features
- um registro do melhor candidato encontrado
