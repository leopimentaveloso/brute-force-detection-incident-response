# 02 — Identificação de Anomalias e Padrões de Ataque

Este documento consolida a interpretação forense dos eventos, estabelecendo a linha divisória entre o comportamento humano rotineiro e a atividade cibernética hostil direcionada à infraestrutura da **TechSolutions Ltda.**

---

## 1. Diferenciação: Ruído Operacional vs. Ameaça Real

Um dos maiores desafios de um analista em um Centro de Operações de Segurança (SOC) é a triagem de falsos positivos. A análise comparativa abaixo fundamenta tecnicamente a separação entre o ruído corporativo e o ataque real:

| Critério de Avaliação | Ruído Operacional (Rede Interna) | Ataque de Força Bruta (IP Externo) |
| :--- | :--- | :--- |
| **Origem do Tráfego** | Sub-rede interna confiável (`192.0.2.0/24`). | IP externo público não autenticado (`203.0.113.50`). |
| **Consistência Usuário / Máquina** | **Alta**: Usuária `m.silva` usa `NOTEBOOK-MARIA`; `a.costa` usa `DESKTOP-ANA`. | **Nula**: Dispositivo registrado como `UNKNOWN`, sem vínculo com colaboradores. |
| **Intervalo Temporal entre Tentativas** | **Humano / Espaçado**: Intervalos de vários minutos (ex: falha às 08:02 e sucesso às 09:20). | **Automatizado / Rápido**: Tentativas a cada **5 a 8 segundos**, humanamente inviáveis sem ferramentas. |
| **Diversidade de Contas Alvo** | Usuários tentam apenas suas próprias credenciais legítimas. | Alternância rápida de múltiplas contas (`administrator`, `m.silva`, `financeiro`, `svc_backup`). |
| **Protocolo e Serviço** | SMB para arquivos internos ou RDP entre estações mapeadas. | RDP direto da internet (`Logon Type 10`) exposto sem VPN ou MFA. |

---

## 2. Caracterização Técnica do Ataque

Com base nos registros forenses, o incidente foi classificado tecnicamente como:

### 2.1. Força Bruta Direcionada com Automação (*Targeted Brute Force & Password Spraying*)
1. **Enumeração Prévia:** O invasor detinha uma lista de usuários plausíveis do ambiente (uma conta administrativa, uma corporativa, uma de setor e uma de serviço).
2. **Uso de Wordlist e Automação:** As rajadas de requisições disparadas com precisão de segundos (ex.: 09:14:03, 09:14:09, 09:14:15, 09:14:22, 09:14:30) evidenciam a utilização de ferramentas automatizadas de autenticação em lote (tais como *Hydra*, *Medusa* ou *Crowbar*).
3. **Vetor de Exploração:** Acesso Remoto Interativo (RDP — Porta padrão TCP 3389) com `Logon Type 10`, aproveitando a ausência de controle de taxa (*rate limiting*) e de bloqueio de conta (*account lockout*).

---

## 3. Ponto Crítico de Investigação: O Comprometimento de Conta

O ponto mais crítico da investigação ocorreu às **09:18:04**:

```csv
2026-11-09 09:18:04,svc_backup,203.0.113.50,RDP,Sucesso,4624,UNKNOWN,Logon type 10; sucesso após falhas; evento suspeito
```

### Anomalias Graves Identificadas neste Evento:
* **Sucesso Imediato após Falhas:** A conta `svc_backup` sofreu falhas consecutivas às 09:17:03 e 09:17:11, obtendo êxito logo na tentativa seguinte (09:18:04).
* **Desvio de Papel Arquitetural:** `svc_backup` é uma **conta de serviço**. Contas dessa natureza executam rotinas em segundo plano e serviços de rede locais (como visto no evento legítimo das 10:10:31, com `Logon Type 3` via SMB). 
* **Violação de Perímetro:** Uma conta de serviço nunca deve possuir privilégio de logon interativo (`Logon Type 10`) nem permissão de autenticação originada de endereços IP externos.

---

## 4. Análise do Comportamento Pós-Comprometimento

Às **10:02:09** e **10:02:16**, o mesmo IP externo (`203.0.113.50`) retomou as tentativas contra o usuário `administrator`.

Isso aponta para dois cenários táticos do atacante:
1. **Tentativa de Escalonamento de Privilégios (*Privilege Escalation*):** Mesmo logado com a conta de backup, o invasor tentou obter acesso total de administrador da máquina/domínio.
2. **Exploração e Persistência:** A credencial de `svc_backup` foi obtida com sucesso e o invasor pode já ter estabelecido persistência ou coletado informações internas antes de reiniciar novos ataques.

---

## 5. Conclusão Forense e Severidade do Incidente

* **Classificação:** Incidente de Segurança Confirmado — **Comprometimento de Credencial de Acesso**.
* **Nível de Severidade:** **ALTO / CRÍTICO**.
* **Justificativa:** O incidente ultrapassou a fase de simples "tentativa" e resultou no acesso indevido efetivo a uma conta com permissões sensíveis no ambiente.
* **Próxima Ação Mandatória:** Acionamento imediato da equipe de Resposta a Incidentes para contenção da conta `svc_backup`, isolamento do host e bloqueio perimetral do IP `203.0.113.50`.
