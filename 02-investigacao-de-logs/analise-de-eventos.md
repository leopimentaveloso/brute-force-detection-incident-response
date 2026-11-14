# 02 — Análise de Eventos de Autenticação e Perímetro

Este documento detalha o levantamento quantitativo, a volumetria de dados e a correlação temporal dos eventos registrados no servidor e no firewall perimetral da **TechSolutions Ltda.** durante o período investigado.

---

## 1. Metodologia de Coleta e Fontes de Dados

Para uma investigação forense íntegra, foram correlacionadas duas camadas distintas de telemetria:

1. **Logs de Host (Autenticação Windows — `logs-autenticacao.csv`):**
   * Registros de auditoria de segurança (`Security.evtx`).
   * **Volume:** 22 eventos (entre 08:02:11 e 10:10:31).
   * **Event ID 4624:** Logon realizado com sucesso.
   * **Event ID 4625:** Falha na tentativa de logon.
2. **Logs de Perímetro (Firewall de Borda — `logs-firewall.csv`):**
   * Registros de descarte de tráfego na borda da rede.
   * **Volume:** 2 eventos (às 10:02:09 e 10:02:16), confirmando a efetividade das medidas de contenção perimetral.

---

## 2. Volumetria Geral de Eventos de Autenticação (Host Windows)

| Métrica | Quantidade | Percentual (%) |
| :--- | :---: | :---: |
| **Total de Eventos no Host (Windows)** | 22 | 100,0% |
| **Falhas de Autenticação (Event ID 4625)** | 17 | 77,3% |
| **Logons com Sucesso (Event ID 4624)** | 5 | 22,7% |

A proporção desbalanceada de falhas (mais de 77%) em um curto intervalo de tempo serviu como o gatilho principal para a abertura da investigação pelo time de segurança.

---

## 3. Distribuição por Origem (Endereço IP no Host)

| Endereço IP | Classificação da Origem | Total no Host | Falhas (4625) | Sucessos (4624) | Dispositivos Associados |
| :--- | :--- | :---: | :---: | :---: | :--- |
| `203.0.113.50` | **Externa (Não autorizada / Atacante)** | **15** | **14** | **1** | `UNKNOWN` |
| `192.0.2.10` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `NOTEBOOK-MARIA` |
| `192.0.2.21` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `NOTEBOOK-JOAO` |
| `192.0.2.35` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `DESKTOP-ANA` |
| `192.0.2.5` | Interna (Servidor de infraestrutura) | 1 | 0 | 1 | `SRV-BACKUP` |

> **Destaque:** O IP externo `203.0.113.50` responde por **68,2% de todas as tentativas** que alcançaram o servidor antes da aplicação do bloqueio perimetral.

---

## 4. Contas Visadas e Vetores de Acesso

### 4.1. Contas visadas pelo IP Externo (`203.0.113.50`) no Servidor
Todas as tentativas externas que atingiram o host foram direcionadas via **RDP (Logon Type 10 - Remote Interactive)** com dispositivo classificado como `UNKNOWN`:

| Conta de Usuário | Tipo de Conta | Falhas | Sucessos | Taxa de Êxito | Status Final |
| :--- | :--- | :---: | :---: | :---: | :--- |
| `administrator` | Administrador local / Domínio | 5 | 0 | 0% | Resistiu ao ataque inicial |
| `m.silva` | Usuário corporativo | 4 | 0 | 0% | Resistiu ao ataque |
| `financeiro` | Conta genérica / Setorial | 3 | 0 | 0% | Resistiu ao ataque |
| `svc_backup` | **Conta de Serviço (Backups)** | **2** | **1** | **33,3%** | **Comprometimento Presumido às 09:18:04** |

---

## 5. Cronologia e Correlação Temporal (Host + Perímetro)

A correlação entre os logs do host e do firewall demonstra a cadência do incidente e a eficácia da contenção:

```text
[08:02 - 08:17] ────────── Atividades legítimas internas com falhas humanas pontuais
                           (m.silva e a.costa digitando senhas incorretas)

[09:14 - 09:18] ────────── 1ª Onda de Ataque (Host Windows - IP 203.0.113.50):
                           • 09:14:03 a 09:14:30 -> 5 rajadas contra 'administrator'
                           • 09:15:02 a 09:15:20 -> 4 rajadas contra 'm.silva'
                           • 09:16:01 a 09:16:15 -> 3 rajadas contra 'financeiro'
                           • 09:17:03 a 09:17:11 -> 2 rajadas contra 'svc_backup'
                           • 09:18:04            -> SUCESSO (4624) em 'svc_backup' [ACESSO SUSPEITO]

[09:19 - 09:25] ────────── Resposta a Incidentes:
                           • Bloqueio perimetral do IP 203.0.113.50 no Firewall.
                           • Queda da sessão ativa, troca de senha e isolamento da conta.

[09:20 - 09:31] ────────── Acessos legítimos internos normais continuam na LAN.

[10:02:09 - 10:02:16] ──── 2ª Onda de Ataque (Logs de Firewall - DROP):
                           • Novas tentativas do IP 203.0.113.50 na porta 3389 descartadas na borda.
                           • O tráfego NÃO chegou ao Windows Server (sem evento 4625 no host).

[10:10:31] ─────────────── Rotina legítima interna de backup via SMB (Logon Type 3) executada.
```
