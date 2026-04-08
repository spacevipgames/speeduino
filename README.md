DETALHES IMPLEMENTAÇÕES 

Resumo Técnico da Implementação (Spool-Sync AE Overlay)

A implementação foi feita como uma camada de transiente sobre o A.E. original, sem criar uma segunda “malha” paralela de combustível. O núcleo continua sendo o correctionAccel() do Speeduino, e o overlay apenas soma autoridade quando detecta dinâmica real de spool.

Arquivos alterados:

transients.cpp
transients.h
corrections.cpp
speeduino.ino
user_moto_profile.h
Como atua (modo de trabalho)

O A.E. base continua normal (TPS-based, com seu blend aeMapWeight).
No final de correctionAccel(), o valor base (accelValue) passa pelo overlay:
AE_final = transientsSpoolAeOverlay(AE_base).
O cálculo pesado do overlay roda só no tick de 30Hz (aprox. 33 ms).
O adder só fica ativo em modo válido:
AE_MODE_TPS e AE_MODE_ADDER,
motor em RUN,
fora de CRANK, fora de DCC, fora de DFCO.
Na montagem do PW, o adder agora entra também quando o overlay está ativo, mesmo sem flag ACC tradicional.
Gatilhos e cálculos

Gatilho de janela de spool:

TPS >= 8.0% (interno >=16 em resolução 0,5%)
MAP >= 90 kPa
Gatilho de rampa:

MAPdot >= 35
e (TPSdot >= 8 ou MAP >= 105 kPa)
Score inteiro:

score = MAPdot*2 + TPSdot*1
Adder do overlay:

spoolAdd = clamp(4 + (score >> 5), 0..28) (%)
soma no AE base:
AE_final = clamp(AE_base + spoolAdd, max 220)
Hold/Decay:

holdTicks = clamp(2 + (score >> 6), max 10) (ticks de 30Hz)
após hold: decai 2% por tick (rampa suave e previsível)
Recovery (troca de marcha / queda de rotação):

janela de recuperação: 12 ticks (~400 ms)
se MAP >= 95 e rpmDOT <= -250, força piso:
spoolAdd >= 8 e hold >= 3
Assist de lambda (opcional, wideband):

se lambda > 1.18, adiciona bônus até +6
se lambda < 0.75, clamp de riqueza em 8
tudo em inteiro (lambda_x100), sem float.
Fusão com A.E. base e blend MAP/TPS

O blend original do A.E. (aeMapWeight) foi mantido.
Em modo TPS, agora mapDOT é sempre calculado (antes só quando aeMapWeight > 0).
tpsDOT efetivo continua podendo ser blendado com mapDOT pelo aeMapWeight.
O overlay usa:
MAPdot direto (dinâmica de carga real),
TPSdot já do fluxo atual (incluindo blend se ativo).
Resultado: você mantém a filosofia do A.E. base, mas ganha autoridade específica em spool/boost transiente.

Custo computacional e “peso”

Aritmética inteira (somas, shifts, clamps); sem float.
Execução principal do overlay limitada a 30Hz.
Estado adicional muito pequeno (poucos bytes estáticos: add/hold/recovery/flags).
Custo extra principal: cálculo de mapDOT sempre no TPS mode (aceitável para ganho de controle em turbo/ITB).
O que espero no comportamento

Menos buraco de torque na entrada de boost.
Menos pico lean no começo do spool.
Menos “tranco” em retomada e pós-troca.
Sem afogar fácil, por causa dos clamps, hold finito e decay controlado.
Como ajustar (ordem técnica recomendada)

Sensibilidade de disparo:
SPOOL_AE_MAPDOT_MIN, SPOOL_AE_TPSDOT_MIN, SPOOL_AE_TPS_MIN
Força:
SPOOL_AE_BASE_ADD, SPOOL_AE_MAX_ADD, SPOOL_AE_TOTAL_MAX
Forma temporal:
SPOOL_AE_HOLD_*, SPOOL_AE_DECAY_STEP
Recuperação de troca:
SPOOL_AE_RECOVERY_*
Refino de mistura:
SPOOL_AE_LAMBDA_*
Tudo está parametrizado em user_moto_profile.h (compile-time), então cada ajuste é determinístico e rastreável por versão.
