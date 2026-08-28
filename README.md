# <p align="center">BOOSTBANDITOS — ENGINEERING ARCHITECTURE</p>

<p align="center">
  Arquitetura técnica da firmware da <a href="https://github.com/spacevipgames/bandit1200turbo">Bandit 1200 Turbo</a>.<br>
  Full Sequential · Alpha-N + MAP Vector · Intelligent Transients · Smart Fuel Pump PWM
</p>

<p align="center">
  <a href="https://github.com/spacevipgames/bandit1200turbo"><img src="https://img.shields.io/badge/PROJECT-BOOSTBANDITOS-cc0000?style=for-the-badge&logo=github" alt="BoostBanditOS"></a>
  <img src="https://img.shields.io/badge/CONTROL-DATA_DRIVEN-045da8?style=for-the-badge" alt="Data Driven">
  <img src="https://img.shields.io/badge/CORE-ATmega2560-111111?style=for-the-badge" alt="ATmega2560">
</p>

---

## Filosofia de controle

BoostBanditOS foi criado com uma regra simples: **cada sinal tem uma função e cada decisão precisa ser visível**.

```text
TPS        → intenção
MAP        → carga
DOT        → velocidade da mudança
Fase       → evento correto
Telemetria → explicação completa
```

O mapa base não é escondido por correções aleatórias. Ele é ampliado por camadas especializadas, com autoridade definida, tempo definido e leitura própria no log.

## Full Sequential: quatro eventos, uma estratégia

O cálculo de combustível preserva PW completo em quatro canais independentes. Cada injetor recebe sua duração no ponto angular correspondente, mantendo a coerência da entrega por cilindro.

```text
PW MODEL → correções → PW FINAL
                          ├─ INJ1
                          ├─ INJ2
                          ├─ INJ3
                          └─ INJ4
```

A arquitetura de sincronismo combina rastreamento de revoluções consecutivas, tolerância a pulso de fase e continuidade de operação. O resultado é uma base sequencial robusta para precisão de combustível e evolução de trim individual.

## Fuel Model: Alpha-N + VE2 MAP Vector

O motor recebe uma leitura composta, perfeita para turbo com ITBs:

```text
VE1 = TPS × RPM
VE2 = MAP × RPM
FUEL = VE1 × VE2
```

VE1 responde imediatamente à borboleta. VE2 atua multiplicativamente como trim da carga real no plenum. Sobre ambas, o Vetor VE2 interpreta crescimento de TPS e MAP, aplicando adição transitória quando a demanda é genuína.

Essa organização separa com elegância:

* intenção do piloto;
* massa de ar real;
* velocidade da entrada de carga;
* combustível final entregue ao cilindro.

## Spool-Sync AE

Spool-Sync AE é um overlay inteligente sobre o AE nativo. A estratégia cruza TPS, MAP, TPSdot, MAPdot, RPMdot e lambda para reconhecer spool, retomada e troca de marcha.

```text
evento de carga → score → adder → hold → decay
```

O sistema oferece ganho por intensidade, hold moldado pelo evento, decay progressivo, recuperação de queda de RPM e sanidade por lambda. A resposta fica cheia quando deve ficar cheia, sem transformar o enriquecimento em uma variável invisível.

## DFCO Reentry: transição de torque

Em ITBs, a volta do combustível define se a moto parece refinada ou brusca. O DFCO Reentry observa MAP, micro-TPS, permanência e lambda para montar uma ponte curta de combustível com score, hold e decay próprios.

O controle entrega uma transição que acompanha o motor e preserva a sensação de conexão direta entre punho e roda.

## Smart Fuel Pump PWM

A bomba é controlada como parte do sistema de combustível. O hardware IDLE1/timer foi convertido em uma arquitetura PWM dedicada que combina:

* pressão PS10;
* RPM, TPS e MAP;
* janelas de operação;
* duty por faixa de pressão;
* pós-partida inteligente;
* clamps e autorização operacional;
* telemetria integrada ao TunerStudio.

Essa camada oferece autoridade hidráulica máxima sob demanda e modulação refinada nas regiões de marcha lenta e baixa carga.

## Total Telemetry: o cálculo aberto

| Domínio        | Dados publicados                                            |
| -------------- | ----------------------------------------------------------- |
| Fuel model     | VE1, VE2, VE2 Base, vetor solicitado e efetivo              |
| Dinâmica       | TPS DOT Raw, MAP DOT Raw, RPMdot, FuelLoad e IgnitionLoad   |
| Enriquecimento | AE nativo, AE híbrido, Spool Add, hold, decay e recuperação |
| DFCO           | reentrada, adição e hold                                    |
| Pulso          | PW Modelo Ar, PW Final, Duty Final e PW por canal           |
| Sistema        | sync, loops/s, loops/rev, avanço e estados de operação      |
| Bomba          | pressão, duty e estado PWM                                  |

O log transforma comportamento em dados. É a fundação para calibrar Moto Profiles específicos — Rua, Pista, Agressivo — e para tornar cada mudança reproduzível.

## Sequential Fuel Trim Matrix

A evolução natural do Full Sequential é o acabamento por cilindro. A **Sequential Fuel Trim Matrix** organiza os trims 1–4 por regiões de RPM, TPS, MAP e carga, permitindo vetores dedicados para:

* baixa carga e marcha lenta;
* aceleração e retomada;
* spool;
* alta carga e boost;
* equalização fina entre cilindros.

```text
Modelo Alpha-N + MAP
          ↓
Trim sequencial 1–4
          ↓
Vetores por zona de operação
          ↓
Entrega individual de combustível
```

Isso transforma o sequencial de recurso de agendamento em ferramenta de acabamento de combustão.

## Princípios de implementação

* matemática inteira, rápida e determinística;
* limites explícitos para cada adder;
* hold e decay no lugar de correções presas;
* baixo uso de CPU/RAM;
* máxima prioridade para fase, ignição e combustível;
* telemetria como parte nativa da arquitetura.

## Origem

**Arquitetura custom:** Marcos Vinicius Estabio

**Projeto principal:** [spacevipgames/bandit1200turbo](https://github.com/spacevipgames/bandit1200turbo)

**Base:** [Speeduino](https://speeduino.com/) e comunidade open source
