# 06 — Relatório Técnico de Incidente de Segurança da Informação

| Documento | Relatório Técnico Forense e de Resposta a Incidentes |
| :--- | :--- |
| **Organização Alvo** | TechSolutions Ltda. |
| **Incidente ID** | INC-20261109-001 |
| **Classificação** | Confidencial / Uso Interno de TI e Segurança |
| **Data do Incidente** | 09 de Novembro de 2026 |
| **Responsável Técnico** | Equipe de Resposta a Incidentes (Blue Team / SOC) |

---

## 1. Sumário do Incidente

Na manhã de 09 de novembro de 2026, entre 09:14:03 e 10:02:16, a infraestrutura da **TechSolutions Ltda.** foi alvo de um ataque cibernético externo direcionado ao serviço de Área de Trabalho Remota (RDP — porta TCP 3389), publicado diretamente na internet.

O atacante utilizou técnicas automatizadas de força bruta por dicionário (*targeted brute force / password spraying*), originadas do endereço IPv4 **`203.0.113.50`**, logrando êxito na obtenção das credenciais da conta de serviço **`svc_backup`** às **09:18:04** (`Event ID 4624`). 

A equipe de segurança identificou o comportamento anômalo, deflagrou o protocolo de contenção em menos de 15 minutos, encerrou as sessões ativas, redefiniu as credenciais comprometidas, isolou logicamente o host e eliminou a exposição perimetral da porta RDP.

---

## 2. Linha do Tempo Técnica dos Eventos

```text
08:02:11 — Ruído Operacional: Falha de logon da usuária legítima m.silva (erro de digitação no NOTEBOOK-MARIA).
08:05:43 — Autenticação legítima com sucesso de j.santos.
08:17:02 — Falha legítima de a.costa via SMB (erro de digitação).
─────────────────────────────────────────────────────────────────────────────────────────────
09:14:03 — [INÍCIO DO ATAQUE] IP 203.0.113.50 dispara 5 tentativas consecutivas contra 'administrator'.
09:15:02 — IP 203.0.113.50 rotaciona alvo para 'm.silva' (4 falhas consecutivas em 18 segundos).
09:16:01 — IP 203.0.113.50 rotaciona alvo para 'financeiro' (3 falhas consecutivas em 14 segundos).
09:17:03 — IP 203.0.113.50 inicia ataque contra 'svc_backup' (2 falhas consecutivas).
09:18:04 — [ACESSO SUSPEITO] IP 203.0.113.50 autentica com SUCESSO (4624, Logon Type 10) em 'svc_backup'.
           Classificado como comprometimento presumido — pendente de validação forense complementar.
─────────────────────────────────────────────────────────────────────────────────────────────
09:19:00 — Detecção da anomalia e acionamento do Comitê de Resposta a Incidentes.
09:25:00 — Queda forçada da sessão RDP, desativação da conta 'svc_backup', redefinição de senha
           e isolamento do servidor em VLAN segura.
─────────────────────────────────────────────────────────────────────────────────────────────
10:02:09 — [LOGS DE FIREWALL] Novas tentativas do IP 203.0.113.50 contra 'administrator' detectadas
10:02:16    no firewall perimetral. O tráfego foi descartado (DROP) antes de atingir o servidor Windows.
           Essas tentativas NÃO geraram eventos 4625 no log de autenticação do Windows — evidência
           de que o bloqueio perimetral estava efetivo desde antes das 10:02.
─────────────────────────────────────────────────────────────────────────────────────────────
10:10:31 — Validação do serviço interno: rotina legítima de backup via SMB (Logon Type 3) executada com sucesso.
```

> **Nota sobre a Distinção de Fontes de Log:** Os eventos de 10:02 estão registrados nos **logs do firewall perimetral** (camada de rede), não nos logs de autenticação do Windows (camada de host). Essa distinção é fundamental na análise forense: logs de host (Windows Security Event Log) só são gerados quando o tráfego chega até o servidor. Logs de borda registram todo o tráfego que tentou entrar na rede, incluindo o que foi bloqueado antes de alcançar qualquer sistema interno. A presença de eventos 4625 apenas até as 09:17 no log do Windows confirma a efetividade do bloqueio perimetral implementado após a detecção.

---

## 3. Vetor de Ataque e Evidências Forenses

1. **Protocolo Explorador:** RDP (*Remote Desktop Protocol*), porta TCP 3389, sem mitigação de NLA (*Network Level Authentication*) ou exigência de canal VPN.
2. **Telemetria de Eventos Windows e Perímetro:**
   * **14 registros com Event ID 4625 (Falha) no host:** Substatus e mensagens indicando senha incorreta repetida sucessivamente originadas de `203.0.113.50` contra as contas `administrator`, `m.silva`, `financeiro` e `svc_backup`.
   * **1 registro com Event ID 4624 (Sucesso) no host:** Confirmação de logon remoto interativo (`Logon Type 10`) às 09:18:04 em `svc_backup`.
   * **2 registros de DROP no Firewall perimetral:** Tentativas às 10:02:09 e 10:02:16 bloqueadas na borda antes de atingir o servidor, confirmando o sucesso da contenção.
3. **Anomalia de Perfil:** A conta `svc_backup` é uma credencial programada para operações locais e de rede (`Logon Type 3`), não devendo possuir direito de logon interativo em console ou via RDP.

---

## 4. Indicadores de Comprometimento (IoCs) e MITRE ATT&CK

* **Endereço IP Atacante:** `203.0.113.50`
* **Porta e Serviço:** TCP 3389 (RDP)
* **Conta com Acesso Suspeito:** `svc_backup` — comprometimento presumido, pendente de validação forense complementar
* **TTPs Mapeadas:**
  * *T1133 (External Remote Services)*
  * *T1110.001 (Password Guessing)*
  * *T1110.003 (Password Spraying)*
  * *T1078 (Valid Accounts)*

---

## 5. Medidas de Contenção, Erradicação e Recuperação

1. **Contenção Perimetral:** Descarte (*DROP*) do endereço IP `203.0.113.50` e revogação imediata das regras de encaminhamento de porta WAN voltadas ao serviço RDP.
2. **Contenção de Identidade:** Execução de comando `logoff` para destruição da sessão remota invasora, desativação preventiva da conta e redefinição de senha com string aleatória de 32 caracteres alfanuméricos e símbolos.
3. **Erradicação:** Verificação completa de processos em execução, chaves de inicialização no Registro do Windows (`HKLM\Software\Microsoft\Windows\CurrentVersion\Run`), tarefas agendadas e ausência de novas contas criadas no sistema.
4. **Recuperação e Validação:** Auditoria de integridade das cópias de segurança (backups), liberação controlada do servidor com monitoramento em tempo real e restrição de acesso administrativo exclusivamente através de túnel criptografado.

---

## 6. Plano de Ação Recomendado (Engenharia de Segurança)

| Prioridade | Ação Técnica | Prazo de Execução | Responsável |
| :---: | :--- | :---: | :--- |
| **P1 (Crítica)** | Implantação de VPN corporativa homologada com autenticação multifator (MFA) obrigatória para todo acesso externo. | Imediato (48h) | Infraestrutura / Redes |
| **P1 (Crítica)** | Configuração de política de restrição de logon interativo para todas as contas de serviço (*Deny log on through Remote Desktop Services* via GPO). | Imediato (24h) | Administração AD / Windows |
| **P2 (Alta)** | Implementação de política de bloqueio de conta (*Account Lockout Policy*) após 3 falhas consecutivas por 30 minutos. | Até 5 dias | TI / Segurança |
| **P2 (Alta)** | Criação de regras de correlação e alerta no SIEM para detecção em tempo real de eventos `4625` múltiplos. | Até 5 dias | Analista de SOC |
| **P3 (Média)** | Condução de testes de intrusão periódicos e auditoria de superfície de ataque externa (ASM). | Semestral | Consultoria / Segurança |
