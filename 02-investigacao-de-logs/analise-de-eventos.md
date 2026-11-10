# 02 — Análise de Eventos de Autenticação

Este documento detalha o levantamento quantitativo, a volumetria de dados e a correlação temporal dos eventos de autenticação registrados no servidor da **TechSolutions Ltda.** durante o período investigado.

---

## 1. Metodologia de Coleta e Fonte de Dados

* **Fonte:** Logs de Auditoria de Segurança do Windows (`Security.evtx` exportado para formato tabular).
* **Período Coberto:** 09 de Novembro de 2026, entre 08:02:11 e 10:10:31.
* **Volume Total:** 24 eventos registrados.
* **Mapeamento de Códigos Windows Event ID:**
  * **Event ID 4624:** Logon realizado com sucesso.
  * **Event ID 4625:** Falha na tentativa de logon.

---

## 2. Volumetria Geral de Eventos

| Métrica | Quantidade | Percentual (%) |
| :--- | :---: | :---: |
| **Total de Eventos Analisados** | 24 | 100,0% |
| **Falhas de Autenticação (Event ID 4625)** | 19 | 79,2% |
| **Logons com Sucesso (Event ID 4624)** | 5 | 20,8% |

A proporção desbalanceada de falhas (quase 80%) em um curto intervalo de tempo serviu como o gatilho principal para a abertura da investigação pelo time de segurança.

---

## 3. Distribuição por Origem (Endereço IP)

Os eventos foram distribuídos em duas origens distintas: rede corporativa interna e endereço IP externo desconhecido.

| Endereço IP | Classificação da Origem | Total de Eventos | Falhas (4625) | Sucessos (4624) | Dispositivos Associados |
| :--- | :--- | :---: | :---: | :---: | :--- |
| `203.0.113.50` | **Externa (Não autorizada / Atacante)** | **17** | **16** | **1** | `UNKNOWN` |
| `192.0.2.10` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `NOTEBOOK-MARIA` |
| `192.0.2.21` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `NOTEBOOK-JOAO` |
| `192.0.2.35` | Interna (Sub-rede corporativa) | 2 | 1 | 1 | `DESKTOP-ANA` |
| `192.0.2.5` | Interna (Servidor de infraestrutura) | 1 | 0 | 1 | `SRV-BACKUP` |

> **Destaque:** O IP externo `203.0.113.50` responde por **70,8% de todas as tentativas** de autenticação registradas no período.

---

## 4. Contas Visadas e Vetores de Acesso

### 4.1. Contas visadas pelo IP Externo (`203.0.113.50`)
Todas as tentativas externas foram direcionadas via **RDP (Logon Type 10 - Remote Interactive)** com dispositivo de origem classificado como `UNKNOWN`:

| Conta de Usuário | Tipo de Conta | Falhas | Sucessos | Taxa de Êxito | Status Final |
| :--- | :--- | :---: | :---: | :---: | :--- |
| `administrator` | Administrador local / Domínio | 7 | 0 | 0% | Resistiu ao ataque inicial |
| `m.silva` | Usuário corporativo | 4 | 0 | 0% | Resistiu ao ataque |
| `financeiro` | Conta genérica / Setorial | 3 | 0 | 0% | Resistiu ao ataque |
| `svc_backup` | **Conta de Serviço (Backups)** | **2** | **1** | **33,3%** | **COMPROMETIDA às 09:18:04** |

---

## 5. Cronologia e Correlação Temporal

A análise temporal revela a dinâmica e a cadência do incidente:

```text
[08:02 - 08:17] ────────── Atividades legítimas internas com falhas humanas pontuais
                           (m.silva e a.costa digitando senhas incorretas)

[09:14 - 09:18] ────────── 1ª Onda de Ataque (IP 203.0.113.50):
                           • 09:14:03 a 09:14:30 -> 5 rajadas contra 'administrator'
                           • 09:15:02 a 09:15:20 -> 4 rajadas contra 'm.silva'
                           • 09:16:01 a 09:16:15 -> 3 rajadas contra 'financeiro'
                           • 09:17:03 a 09:17:11 -> 2 rajadas contra 'svc_backup'
                           • 09:18:04            -> SUCESSO (4624) em 'svc_backup' [BRECHA]

[09:20 - 09:31] ────────── Retorno de acessos legítimos internos normais

[10:02:09 - 10:02:16] ──── 2ª Onda de Ataque (IP 203.0.113.50):
                           • Novas tentativas automatizadas contra 'administrator'
```
