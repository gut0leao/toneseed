# Specs

Esta pasta contem especificacoes versionadas do ToneSeed.

Cada spec deve transformar uma ideia em um contrato revisavel antes da implementacao.

## Quando Criar Uma Spec

Crie uma spec quando a mudanca:

- adiciona comportamento novo
- define contratos de dados
- altera arquitetura
- envolve hardware
- precisa de criterios de aceite
- pode gerar ambiguidade para IA ou humanos

Mudancas pequenas de texto ou correcao de typo nao precisam de spec.

## Estrutura Recomendada

```text
specs/
    0001-nome-curto/
        spec.md
```

## Status Possiveis

- `Draft`: ainda em elaboracao
- `Ready`: pronta para implementacao
- `In Progress`: implementacao em andamento
- `Done`: implementada e validada
- `Superseded`: substituida por outra spec

## Checklist De Uma Boa Spec

- problema claro
- objetivo explicito
- fora de escopo
- comportamento esperado
- contratos de dados ou comandos
- criterios de aceite
- plano de validacao
- riscos e decisoes abertas

## Relacao Com Codigo

A ordem desejada e:

```text
spec -> testes -> codigo -> validacao -> atualizacao da spec/backlog
```

Quando isso nao for pratico, registrar o motivo na propria spec ou no commit.
