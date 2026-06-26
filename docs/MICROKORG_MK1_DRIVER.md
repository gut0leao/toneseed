# microKORG Mk1 Driver

> Architectural update: this document describes a future hardware target driver. The first validatable ToneSeed prototype should use the Virtual Synth MVP plus separate `SynthDriver` and `SynthRuntime` abstractions before adding microKORG-specific MIDI/SysEx behavior.

## Objetivo

Definir o escopo do driver de hardware para o Korg microKORG Mk1.

O driver e responsavel por traduzir a Tone IR e parametros internos do ToneSeed para mensagens compreensiveis pelo microKORG Mk1.

## Responsabilidades Do Driver

- Listar parametros suportados.
- Validar faixas de valores.
- Converter parametros normalizados para valores nativos do synth.
- Enviar notas MIDI.
- Enviar control changes quando aplicavel.
- Enviar e receber mensagens SysEx quando necessario.
- Registrar patches gerados.
- Proteger o usuario contra sobrescrita acidental de patches importantes.

## Fora Da Responsabilidade Do Driver

- Analisar audio.
- Decidir a similaridade entre timbres.
- Executar o loop de otimizacao.
- Capturar audio.
- Construir interface grafica.

## Camadas Sugeridas

```text
Tone IR
      ↓
Mapping heuristico
      ↓
Patch normalizado
      ↓
microKORG Mk1 Driver
      ↓
MIDI / SysEx
      ↓
microKORG Mk1
```

## Modelo De Parametro Normalizado

Internamente, o ToneSeed pode representar parametros em escala normalizada de `0.0` a `1.0`.

Exemplo:

```json
{
  "osc1_wave": "saw",
  "osc2_level": 0.35,
  "filter_cutoff": 0.78,
  "filter_resonance": 0.22,
  "amp_attack": 0.08,
  "amp_decay": 0.45,
  "amp_sustain": 0.60,
  "amp_release": 0.30,
  "lfo1_rate": 0.18,
  "lfo1_depth": 0.12
}
```

O driver converte esses valores para o formato nativo do microKORG.

## Parametros Iniciais Candidatos

O driver do microKORG deve comecar com um subconjunto pequeno.

- voice mode
- oscillator 1 waveform
- oscillator 1 control values
- oscillator 2 waveform
- oscillator 2 tuning
- oscillator balance
- noise level
- filter type
- filter cutoff
- filter resonance
- filter envelope amount
- amp level
- amp envelope attack
- amp envelope decay
- amp envelope sustain
- amp envelope release
- filter envelope attack
- filter envelope decay
- filter envelope sustain
- filter envelope release
- LFO 1 waveform
- LFO 1 rate
- LFO 1 intensity
- LFO 2 waveform
- LFO 2 rate
- modulation routing
- effect type
- effect depth

## Estrategia De Implementacao

### Fase 1: MIDI Basico

- Detectar portas MIDI.
- Abrir porta de saida.
- Enviar `note_on`.
- Esperar duracao configurada.
- Enviar `note_off`.
- Permitir escolha de canal MIDI.

### Fase 2: Leitura E Backup

- Receber dados do microKORG.
- Investigar dumps SysEx.
- Salvar backup de patch ou banco antes de escrever.
- Registrar dumps brutos para analise.

### Fase 3: Escrita Controlada

- Enviar alteracoes de parametro com valores conhecidos.
- Validar no synth se a alteracao foi aplicada.
- Evitar sobrescrever bancos inteiros no inicio.
- Definir um slot seguro para testes.

### Fase 4: Patch Completo

- Construir uma representacao completa de patch.
- Serializar patch para JSON.
- Converter patch para SysEx.
- Enviar patch ao microKORG.
- Tocar e capturar automaticamente.

## Arquivos De Dados Sugeridos

```text
toneseed/
    drivers/
        microkorg_mk1.py

assets/
    synths/
        microkorg_mk1/
            parameters.json
            sysex.md
            default_patch.json
            safe_test_patch.json
```

## `parameters.json`

O mapa de parametros deve ser declarativo.

Exemplo:

```json
{
  "filter_cutoff": {
    "label": "Filter Cutoff",
    "type": "continuous",
    "normalized_min": 0.0,
    "normalized_max": 1.0,
    "native_min": 0,
    "native_max": 127,
    "unit": "midi_value"
  }
}
```

Quando os enderecos SysEx forem confirmados, o mesmo arquivo pode incluir informacoes especificas:

```json
{
  "filter_cutoff": {
    "label": "Filter Cutoff",
    "native_min": 0,
    "native_max": 127,
    "sysex_address": "TBD",
    "sysex_format": "TBD"
  }
}
```

## Cuidados Com SysEx

- Confirmar o modelo exato antes de enviar dumps.
- Usar canal MIDI correto.
- Fazer backup antes de escrever patches.
- Comecar por leitura antes de escrita.
- Registrar bytes SysEx brutos em arquivos de teste.
- Comparar dumps antes/depois de alterar um parametro manualmente.

## Metodo Para Descobrir Parametros

Um metodo pratico para documentar o SysEx:

1. Criar ou selecionar um patch de teste.
2. Fazer dump SysEx do patch.
3. Alterar um unico parametro no microKORG.
4. Fazer novo dump SysEx.
5. Comparar bytes alterados.
6. Repetir para cada parametro relevante.
7. Registrar resultados em `assets/synths/microkorg_mk1/sysex.md`.

## Testes Do Driver

Testes sem hardware:

- validar conversao de valores normalizados
- validar limites de parametros
- validar serializacao JSON
- validar construcao de mensagens MIDI

Testes com hardware:

- detectar portas
- tocar nota
- receber dump
- enviar parametro simples
- enviar patch seguro
- capturar audio gerado

## Criterio De Pronto

O driver sera considerado utilizavel no MVP quando permitir:

- selecionar porta MIDI
- tocar nota no microKORG Mk1
- representar patch em JSON
- converter parametros normalizados para valores nativos
- enviar um conjunto minimo de parametros ao synth
- capturar audio resultante em outro modulo do ToneSeed
