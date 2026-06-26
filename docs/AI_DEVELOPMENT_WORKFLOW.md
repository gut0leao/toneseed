# AI Development Workflow

## Objetivo

Definir como o ToneSeed sera desenvolvido com apoio de IA sem perder controle arquitetural.

Este projeto usa um fluxo spec-driven simples, com um unico agente Codex, specs versionadas e validacao explicita antes de considerar uma tarefa pronta.

## Principios

- Especificar antes de implementar.
- Manter contexto do projeto dentro do repositorio.
- Transformar decisoes importantes em documentos rastreaveis.
- Fazer a IA executar dentro de limites claros.
- Validar resultado com testes, comandos ou criterios objetivos.
- Preferir entregas pequenas e revisaveis.

## Fluxo De Trabalho

```text
Ideia
  ↓
Spec
  ↓
Criterios de aceite
  ↓
Plano tecnico curto
  ↓
Implementacao
  ↓
Validacao
  ↓
Registro do resultado
```

## Estrutura Dos Artefatos

```text
docs/
    AI_DEVELOPMENT_WORKFLOW.md
    MVP_MICROKORG_MK1.md
    HARDWARE_SETUP.md
    MICROKORG_MK1_DRIVER.md
    adr/
        0001-use-python-for-mvp.md

specs/
    README.md
    templates/
        feature-spec.md
    0001-microkorg-mk1-mvp/
        spec.md
    0002-virtual-synth-mvp/
        spec.md
```

## Como Pedir Trabalho Ao Codex

Cada pedido de implementacao deve referenciar uma spec ou pedir a criacao de uma spec antes do codigo.

Exemplos:

```text
Implemente a primeira parte da spec specs/0001-microkorg-mk1-mvp/spec.md: deteccao de portas MIDI.
```

```text
Crie uma spec para captura de audio pela Fast Track C400 antes de implementar.
```

```text
Revise a spec 0001 e diga quais criterios de aceite ainda nao tem validacao automatizada.
```

## Definicao De Pronto

Uma tarefa so deve ser considerada pronta quando tiver:

- spec ou documento de referencia
- criterio de aceite claro
- codigo ou documento alterado
- validacao executada ou motivo registrado para nao executar
- status refletido no backlog, quando aplicavel

## Niveis De Spec

### Spec De Produto

Descreve o comportamento desejado do ponto de vista do usuario.

Example: "Generate a patch inspired by a WAV and render it through a controllable virtual synth."

### Spec Tecnica

Descreve contratos, modulos, comandos, dados, validacoes e limites.

Exemplo: "Enviar `note_on` e `note_off` pela porta MIDI selecionada".

### ADR

Registra uma decisao arquitetural que deve continuar sendo respeitada.

Exemplo: "Python sera a linguagem principal do MVP".

## Contratos Executaveis

Sempre que possivel, uma spec deve virar um ou mais testes.

Exemplos:

- conversao de nota `C3` para numero MIDI
- validacao de parametros normalizados entre `0.0` e `1.0`
- serializacao da Tone IR em JSON
- selecao de dispositivos MIDI por nome
- deteccao de clipping em audio capturado

## Validacao Inicial Do Projeto

Enquanto nao houver codigo, a validacao e documental:

```text
rg --files
git status --short --branch
```

Quando o pacote Python existir, a validacao minima devera ser:

```text
pytest
python -m toneseed --help
```

When the virtual synth path exists, validation should first prove the minimum loop without real hardware:

```text
toneseed synths
toneseed play-note --synth surge-xt --note C3 --duration 1
toneseed grow examples/audio/target.wav --synth surge-xt
```

When hardware support exists later, validation with hardware should be manual and recorded:

```text
toneseed devices
toneseed play-note --note C3 --duration 1
toneseed capture --note C3 --duration 2 --output captures/c3.wav
```

## Regras Para Evolucao

- Do not add real hardware support before validating the minimum cycle with a controllable virtual synth.
- Nao introduzir machine learning antes de existir captura e comparacao de features.
- Nao criar interface grafica antes de existir CLI funcional.
- Preserve the separation between audio analysis, Tone IR, normalized patches, synth-specific drivers, and synth runtimes.
- Nao misturar analise de audio com detalhes de SysEx, MIDI CC, plugin hosting, or synth automation.
- Nao sobrescrever patches do microKORG sem uma rotina de backup documentada.

## Uso De Um Unico Agente

Este projeto nao depende de orquestracao com multiplos agentes.

O Codex deve atuar como executor unico, seguindo os documentos do repositorio:

1. ler a spec relevante
2. inspecionar o codigo existente
3. propor ou executar uma mudanca pequena
4. validar
5. atualizar documentos quando a decisao mudar
