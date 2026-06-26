# Hardware Setup

> Architectural update: this document is for the later microKORG hardware target. The first validatable ToneSeed prototype should use the Virtual Synth MVP before depending on physical MIDI/audio wiring and external capture.

## Objetivo

Documentar a configuracao fisica inicial para desenvolver e testar o ToneSeed com o Korg microKORG Mk1 e a interface M-Audio Fast Track C400.

## Equipamentos

- Computador
- M-Audio Fast Track C400
- Korg microKORG Mk1
- Fonte de alimentacao do microKORG
- Cabo USB da interface
- Dois cabos MIDI DIN 5 pinos
- Um ou dois cabos P10 para audio
- Fones ou monitores

## Conexoes MIDI

```text
Fast Track C400 MIDI Out
      ↓
microKORG MIDI In

microKORG MIDI Out
      ↓
Fast Track C400 MIDI In
```

O caminho `Fast Track MIDI Out -> microKORG MIDI In` permite que o ToneSeed toque notas e envie mensagens para o synth.

O caminho `microKORG MIDI Out -> Fast Track MIDI In` permite receber dumps, respostas SysEx e mensagens enviadas pelo synth.

## Conexoes De Audio

Configuracao mono inicial:

```text
microKORG Output L/Mono
      ↓
Fast Track C400 Line Input
```

Configuracao stereo opcional:

```text
microKORG Output L
      ↓
Fast Track C400 Input 1

microKORG Output R
      ↓
Fast Track C400 Input 2
```

Para o MVP de hardware do microKORG, mono e suficiente e simplifica a analise.

## Configuracao Inicial Recomendada

- Usar saida `L/Mono` do microKORG.
- Entrar na interface em nivel de linha, nao em modo instrumento, se a interface permitir essa escolha.
- Desativar efeitos externos ou processamento do sistema operacional.
- Manter volume do microKORG em nivel moderado.
- Ajustar ganho da interface para evitar clipping.
- Gravar em 44.1 kHz ou 48 kHz.
- Usar 24-bit se disponivel.

## Teste Manual Antes Do Software

Antes de executar qualquer automacao:

1. Conectar o microKORG na interface.
2. Abrir uma DAW ou gravador de audio.
3. Tocar o microKORG manualmente.
4. Confirmar que o audio chega sem clipping.
5. Confirmar que o canal MIDI de entrada do microKORG esta correto.
6. Enviar uma nota MIDI simples por algum software ou DAW.
7. Confirmar que o microKORG toca a nota recebida.

## Teste Automatizado Basico

O primeiro teste de hardware do ToneSeed deve apenas enviar uma nota MIDI e capturar audio.

```text
toneseed play-note --note C3 --velocity 100 --duration 2
toneseed capture --note C3 --duration 2 --output captures/c3.wav
```

Esse teste valida:

- porta MIDI de saida
- canal MIDI
- audio input
- taxa de amostragem
- ganho
- ausencia de clipping

## Calibracao

O ToneSeed deve ter uma rotina de calibracao para medir:

- latencia entre envio MIDI e audio capturado
- nivel medio do sinal
- clipping
- ruido de fundo
- tempo de release audivel

Exemplo de comando futuro:

```text
toneseed calibrate --synth microkorg-mk1
```

## Boas Praticas

- Criar uma pasta `captures/` para gravacoes geradas automaticamente.
- Criar uma pasta `calibration/` para arquivos de medicao.
- Salvar metadados de cada captura em JSON.
- Nao sobrescrever patches importantes no microKORG durante testes.
- Usar um banco ou slot dedicado para experimentos.

## Solucao De Problemas

Se o microKORG nao tocar:

- verificar se o cabo MIDI esta no sentido correto
- verificar canal MIDI
- verificar se a porta MIDI correta foi selecionada
- testar com uma DAW antes do ToneSeed

Se nao houver audio capturado:

- verificar se a saida L/Mono esta conectada
- verificar ganho da interface
- verificar selecao do input no sistema
- testar gravacao em uma DAW

Se houver clipping:

- reduzir volume master do microKORG
- reduzir ganho de entrada da interface
- repetir captura de teste
