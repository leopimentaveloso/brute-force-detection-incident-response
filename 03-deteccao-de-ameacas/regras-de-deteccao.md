# 03 — Regras de Detecção e Políticas Preventivas

Este documento estabelece as regras de monitoramento contínuo (SIEM) e as políticas preventivas que devem ser implementadas na **TechSolutions Ltda.** para detectar e mitigar tentativas de força bruta e comprometimento de contas antes que o invasor obtenha sucesso.

---

## 1. Regras Lógicas de Detecção (SIEM / SOC)

### Regra 01: Detecção de Força Bruta / Password Spraying via RDP
* **Objetivo:** Identificar ferramentas automatizadas de ataque por dicionário disparadas contra serviços remotos.
* **Gatilho Lógico:**
  * `EventID == 4625` (Falha de Logon)
  * `LogonType == 10` (RemoteInteractive - RDP)
  * **Agrupamento:** Por `IpAddress` de origem
  * **Limite (*Threshold*):** $\ge 3$ tentativas falhas em um intervalo de até 5 minutos.
* **Severidade:** **ALTA**.
* **Ação Automatizada:** Disparo de alerta imediato no SOC e inclusão temporária do IP em lista de bloqueio (*shun/block list*) no Firewall de borda.

### Regra 02: Alerta de Desvio de Função em Contas de Serviço
* **Objetivo:** Detectar uso indevido ou anômalo de credenciais não humanas.
* **Gatilho Lógico:**
  * `TargetUserName` inicia com `svc_` ou pertence ao grupo `Contas de Serviço`.
  * `LogonType == 10` (Tentativa de logon interativo/visual) OU `IpAddress` NÃO pertencente à rede interna corporativa (`192.0.2.0/24`).
* **Severidade:** **CRÍTICA (P1)**.
* **Ação:** Bloqueio instantâneo da tentativa e acionamento de plantão de segurança.

---

## 2. Regra em Padrão Aberto (Sigma Rule)

O formato **Sigma** é o padrão universal da indústria para documentar assinaturas de detecção agnósticas de ferramenta (podendo ser convertidas para Splunk, Elastic, Microsoft Sentinel, Wazuh, etc.):

```yaml
title: Tentativa de Força Bruta RDP de Origem Externa
id: 7c5a3d4e-128b-4b21-9988-a6b1234ef012
status: experimental
description: >
  Detecta múltiplas falhas de autenticação RDP (Event ID 4625, Logon Type 10)
  originadas de um mesmo IP externo em curto intervalo de tempo.
  Deve ser convertida para a linguagem específica do SIEM alvo (Splunk SPL,
  Elastic EQL, Microsoft Sentinel KQL, Wazuh, etc.) e validada em ambiente
  controlado antes de promover para status 'test' ou 'stable'.
author: Analista de Seguranca / TechSolutions
date: 2026/11/09
modified: 2026/11/09
references:
  - https://attack.mitre.org/techniques/T1110/
  - https://docs.sigmahq.io/docs/basics/rules.html
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4625
    LogonType: 10
  filter_internal_ips:
    IpAddress|startswith:
      - '192.168.'
      - '10.'
      - '172.16.'
      - '192.0.2.'   # bloco fictício usado neste laboratório (RFC 5737)
  condition: selection and not filter_internal_ips
# Nota: a agregação por threshold (>= 3 eventos / 5 min / mesmo IP) deve ser
# implementada como regra de correlação no SIEM alvo, pois o formato Sigma base
# não suporta nativalmente contagem por janela de tempo na condition.
# Exemplo para Splunk SPL:
#   index=wineventlog EventCode=4625 Logon_Type=10 NOT [blocklist de IPs internos]
#   | bin _time span=5m | stats count by IpAddress, _time | where count >= 3
fields:
  - IpAddress
  - TargetUserName
  - WorkstationName
  - LogonType
falsepositives:
  - Colaboradores em home office digitando senha incorreta consecutivamente
    (risco mitigado pela adoção de VPN obrigatória antes do acesso RDP).
level: high
tags:
  - attack.credential_access
  - attack.t1110
  - attack.t1110.001
  - attack.t1133
```

---

## 3. Diretivas e Controles Preventivos (*Hardening*)

1. **Política de Bloqueio de Contas (*Account Lockout Policy*):**
   * Bloqueio temporário da conta após **3 a 5 tentativas inválidas** consecutivas por um período de 15 a 30 minutos.
2. **Atraso Exponencial (*Exponential Backoff*):**
   * Inserção de latência progressiva entre erros consecutivos para inviabilizar o uso de wordlists de alta velocidade.
3. **Restrição Perimetral de Acesso Remoto:**
   * Fechamento da porta RDP (TCP 3389) exposta diretamente para a internet.
   * Exigência obrigatória de túnel **VPN corporativa** ou **Zero Trust Network Access (ZTNA)** para colaboradores externos.
4. **Autenticação Multifator (MFA / 2FA):**
   * Obrigatoriedade de segundo fator em todas as autenticações de acesso remoto corporativo.
