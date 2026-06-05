# nrf-forge

Desenvolva firmware para o **nRF54LM20B** com **nRF Connect SDK v3.3.0 + Zephyr** inteiramente via prompts — sem escrever C, sem tocar em ponteiro. Inspirado no [vgv-wingspan](https://github.com/VeryGoodOpenSource/vgv-wingspan), adaptado para embarcados.

## Filosofia

1. **Você descreve o comportamento, o Claude escreve o C.** Toda decisão é explicada em termos de produto (bateria, alcance, latência) — nunca em jargão de memória.
2. **Workflow com artefatos**: brainstorm → plano → build → review, cada fase salva um documento em `docs/` para a próxima fase continuar com contexto limpo.
3. **Quality gate obrigatório**: todo código C passa por dois agentes de revisão (segurança de memória + padrões Zephyr) antes de ser considerado pronto. O agente de memória é o seu seguro contra ponteiros.
4. **Hardware atrás de devicetree**: tudo configurado por overlay/Kconfig, nunca editando o SDK — o que torna barato o porte futuro do DK para a placa custom.

## Workflow (4 fases)

| Skill | O que faz |
|---|---|
| `fw-brainstorm` | Explora requisitos de uma feature em diálogo → `docs/brainstorm/` |
| `fw-plan` | Transforma em plano passo-a-passo com Kconfig/devicetree/módulos → `docs/plan/` |
| `fw-build` | Implementa o plano: código + `west build` a cada passo + agentes de revisão |
| `fw-review` | Revisão de qualidade sob demanda (memória + Zephyr) → `docs/reviews/` |

## Skills técnicas

| Skill | O que faz |
|---|---|
| `nrf-new-project` | Scaffold de app NCS 3.3.0 com sysbuild e primeiro build limpo |
| `nrf-build-flash` | Build, flash, logs RTT e troubleshooting de erros |
| `nrf-devicetree` | Sensores e periféricos (I2C, SPI, ADC, PDM, GPIO) via overlay |
| `nrf-ble` | BLE periférico (app de config) + central (rede fog), GATT, segurança |
| `nrf-dfu` | DFU/OTA seguro: MCUboot, assinatura ED25519, chaves no KMU |
| `nrf-power` | Orçamento de energia, duty cycle, suspensão de periféricos, PPK2 |
| `nrf-custom-board` | Board definition própria e sequência de bring-up da placa final |
| `nrf-edge-ai` | ML on-device: NPU Axon, FLPR, TFLite Micro, pipeline de dados |

## Agentes

- `memory-safety-review-agent` — caça buffer overflow, NULL, use-after-free, stack overflow, violações de ISR
- `zephyr-review-agent` — devicetree/Kconfig corretos, threading, logging, consumo
- `ncs-research-agent` — pesquisa docs oficiais Nordic/Zephyr na versão exata do SDK

## Alvo fixado

- SoC: nRF54LM20B (Cortex-M33 @128 MHz + FLPR RISC-V + NPU Axon, 2 MB RRAM, 512 KB RAM, BLE 6.0)
- Board target: `nrf54lm20dk/nrf54lm20b/cpuapp` (DK agora; placa custom depois)
- SDK: nRF Connect SDK v3.3.0, build sempre com sysbuild
- Produto: dispositivo de parede a bateria (USB-C para carga) medindo pressão sonora, IAQ e iluminância; Edge AI extensiva; BLE com rede fog + app de configuração; gateway NB-IoT para a cloud

## Como começar

Com o plugin instalado e o ambiente NCS pronto, diga por exemplo:

> "cria o projeto do firmware" → `nrf-new-project`
> "quero fazer o brainstorm da medição de IAQ" → `fw-brainstorm`

## Extensões futuras

- `.mcp.json` com context7 (docs ao vivo) e Edge Impulse MCP (treino de modelos via prompt)
- Skill de NB-IoT quando o módulo/modem do gateway for definido
