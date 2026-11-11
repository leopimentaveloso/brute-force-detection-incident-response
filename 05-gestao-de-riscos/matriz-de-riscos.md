# 05 — Gestão de Riscos, Controles ISO/IEC 27001 e Parecer DPO (LGPD)

Este documento consolida a avaliação de impacto e riscos corporativos decorrentes do incidente de segurança ocorrido na **TechSolutions Ltda.**, estabelecendo o enquadramento na norma internacional **ISO/IEC 27001:2022** e as obrigações regulatórias sob a ótica da **LGPD (Lei nº 13.709/2018)**.

---

## 1. Avaliação de Riscos (Matriz 5x5)

A metodologia adotada mensura a **Probabilidade (1 a 5)** versus o **Impacto (1 a 5)** na tríade fundamental da segurança da informação (Confidencialidade, Integridade e Disponibilidade — CID):

| Dimensão CID | Situação Observada no Incidente | Nível de Afetação |
| :--- | :--- | :---: |
| **Confidencialidade** | **Comprometimento Presumido:** O invasor obteve autenticação válida na conta `svc_backup` (forte evidência de acesso não autorizado, pendente de validação forense complementar quanto à extensão da atividade pós-autenticação). | **Crítico** |
| **Integridade** | **Sob Investigação:** As ações pós-comprometimento foram estancadas nos primeiros minutos; não foram constatadas adulterações de arquivos ou alterações de privilégios. | **Médio** |
| **Disponibilidade** | **Preservada:** Os serviços e a infraestrutura permaneceram operacionais e não sofreram ataques de negação de serviço (DoS) ou sequestro por ransomware. | **Baixo** |

### Comparativo de Níveis de Risco:
* **Risco Inerente (Sem controles adequados):**
  * Probabilidade: 5 (Quase Certo — RDP aberto sem bloqueio na internet pública)
  * Impacto: 5 (Catastrófico — Comprometimento de credenciais de serviço)
  * **Score:** $5 \times 5 = \mathbf{25}$ (**Risco Extremo / Crítico**).
* **Risco Residual (Após contenção, MFA e isolamento por VPN):**
  * Probabilidade: 1 (Raro — Acesso restrito a VPN com segundo fator e bloqueio automático)
  * Impacto: 2 (Menor — Contenção imediata sem privilégios administrativos concedidos)
  * **Score:** $1 \times 2 = \mathbf{2}$ (**Risco Baixo / Aceitável**).

---

## 2. Mapeamento de Falhas e Controles da ISO/IEC 27001:2022

O incidente evidenciou não conformidades e oportunidades diretas de implementação dos seguintes controles do **Anexo A da ISO/IEC 27001:2022**:

| Código ISO 27001 | Nome do Controle | Falha Identificada na TechSolutions | Ação Corretiva Mandatória |
| :--- | :--- | :--- | :--- |
| **A.5.15** | *Controle de Acesso* | Ausência de diretiva de segregação de funções e logon interativo liberado para contas de serviço. | Configurar *Deny log on through Remote Desktop Services* para todas as contas de serviço (`svc_*`). |
| **A.8.5** | *Autenticação Segura* | Ausência de política de bloqueio de conta (*Account Lockout*) e ausência de MFA para acessos externos. | Implementar bloqueio após 3 tentativas inválidas consecutivas e impor MFA corporativo. |
| **A.8.2** | *Direitos de Acesso Privilegiados* | A conta de rotina de backup possuía direitos de logon interativo (RDP) desnecessários e vetor de autenticação acessível pela internet pública. | Isolar contas privilegiadas em cofres de senhas (PAM) e restringir conexões exclusivamente à LAN interna, negando explicitamente direitos de logon remoto interativo. |
| **A.8.20** | *Segurança de Redes* | Porta RDP (TCP 3389) publicada diretamente na WAN sem camada de filtragem intermediária. | Fechar publicação direta de portas administrativas na internet e canalizar acessos via VPN com túnel criptografado. |
| **A.5.24** | *Planejamento e Preparação para Gestão de Incidentes* | Detecção inicial dependente de análise manual da equipe de TI após o sucesso do atacante. | Estabelecer regras automatizadas no SIEM para alerta e bloqueio preventivo de força bruta em tempo real. |

---

## 3. Parecer Consultivo do Encarregado de Dados (DPO) — LGPD

* **Base Legal:** Lei Geral de Proteção de Dados Pessoais (**Lei nº 13.709/2018**), especificamente os artigos 46 e 48, complementados pela **Resolução CD/ANPD nº 15/2024**.
* **Ativo Acometido:** A conta violada (`svc_backup`) possui permissões de leitura e gravação em repositórios de backup corporativos, os quais abrigam bases de dados de clientes, fornecedores e folha de pagamento de colaboradores da TechSolutions Ltda.

### 3.1. Avaliação do Risco Relevante (Art. 48, LGPD)
1. **Configuração de Risco Relevante:** Embora a contenção tenha ocorrido em menos de 15 minutos e não haja evidência material de exfiltração em massa de dados, o acesso ilegítimo a uma credencial de custódia de backups corporativos configura risco potencial concreto à confidencialidade dos titulares.
2. **Princípio da Segurança e da Prestação de Contas (Art. 6º, VIII e X):** A empresa tem o dever de demonstrar diligência e transparência perante a autoridade fiscalizadora e a sociedade.

### 3.2. Cenários de Dano Concreto aos Titulares e Riscos Secundários
* **Golpes Governamentais e Cobranças Falsas:** A obtenção de dados cadastrais (CPF, endereço, e-mail, telefone) permite que criminosos elaborem campanhas de engenharia social altamente direcionadas (*spear phishing*), passando-se por **órgãos do governo (Receita Federal, INSS, Tribunais e Cartórios)** para exigir pagamentos e boletos falsos dos clientes da empresa.
* **Ameaça de Ransomware e Extorsão por Criptoativos:** O acesso ao servidor viabilizaria a implantação de ransomware para criptografia em massa de arquivos e backups, seguido de extorsão com pedidos de resgate em criptomoedas. A literatura forense demonstra que o pagamento do resgate frequentemente **não resulta na chave de descriptografia** e alimenta redes criminosas.
* **Dano Reputacional e Perda de Mercado:** O descrédito institucional perante parceiros e clientes compromete a viabilidade operacional e comercial do negócio a médio e longo prazo.

### 3.3. Medidas Obrigatórias Determinadas pelo DPO:
* **Comunicação Preliminar à ANPD:** Protocolar notificação de incidente de segurança no portal eletrônico da ANPD no prazo regulamentar de **3 (três) dias úteis**, contados do conhecimento do fato, informando a natureza dos dados potencialmente expostos e as medidas de mitigação executadas.
* **Comunicação aos Titulares de Dados:** Redigir comunicado formal, em linguagem clara e acessível, alertando os titulares sobre as medidas de segurança preventivas adotadas pela empresa e disponibilizando canal de atendimento direto com o DPO.
* **Registro Interno do Incidente:** Catalogar detalhadamente o evento no Livro de Registro de Incidentes de Segurança da Informação da organização, servindo de evidência de conformidade em auditorias futuras.
