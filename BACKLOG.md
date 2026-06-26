# Backlog

Backlog inicial orientado por specs.

## Now

- [x] Create `specs/0002-virtual-synth-mvp/spec.md`.
- [ ] Define the `SynthDriver` contract.
- [ ] Define the `SynthRuntime` contract.
- [ ] Define the Tone IR -> normalized patch mapping.
- [ ] Criar esqueleto Python do pacote `toneseed`.
- [ ] Implementar conversao de notas musicais para numeros MIDI.
- [ ] Implement an initial virtual synth driver.
- [ ] Document the Carla runtime setup path for the MVP.

## Next

- [ ] Implementar Tone IR serializavel.
- [ ] Implementar `toneseed analyze`.
- [ ] Criar exemplo minimo de audio para testes automatizados.
- [ ] Test note MIDI/control delivery to the virtual synth.
- [ ] Define routed audio capture for the Carla-hosted synth output.
- [ ] Create a simple comparison between target audio and generated audio.

## Later

- [ ] Resolver perguntas abertas da `specs/0001-microkorg-mk1-mvp/spec.md`.
- [ ] Implementar comando `toneseed devices` para hardware real.
- [ ] Implementar `toneseed play-note` para hardware real.
- [ ] Implementar captura WAV por interface de audio externa.
- [ ] Documentar SysEx do microKORG Mk1 a partir de dumps reais.
- [ ] Implementar driver minimo do microKORG Mk1.
- [ ] Implementar primeiro loop de otimizacao heuristica.
- [ ] Criar UI depois da CLI estar funcional.
