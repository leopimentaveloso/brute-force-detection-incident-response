# Detecção de Força Bruta e Resposta a Incidentes (Brute Force Detection & Incident Response)

Projeto prático de segurança defensiva (*Blue Team*), investigação forense em registros de autenticação do Windows e estruturação de resposta a incidentes alinhada às melhores práticas de governança e segurança da informação (**ISO/IEC 27001** e **NIST SP 800-61**).

---

## 🌐 Idiomas / Languages
[🇧🇷 Português (Original)](README.md) | [🇺🇸 English](README.en.md) | [🇪🇸 Español](README.es.md)

---

## 🎯 Objetivo do Projeto

Simular, identificar, conter e documentar um ataque de força bruta (*Brute Force / Password Spraying*) contra serviços de acesso remoto (RDP), avaliando o comprometimento de credenciais corporativas e estabelecendo planos de ação técnicos e executivos.

Este projeto complementa o laboratório prático de exploração ofensiva com *Medusa*, trazendo a perspectiva do **defensor** (SOC / Blue Team / GRC).

---

## 🏢 Cenário Fictício: TechSolutions Ltda.

* **Organização:** TechSolutions Ltda.
* **Segmento:** Prestação de serviços de tecnologia e suporte empresarial.
* **Porte:** Aproximadamente 50 colaboradores.
* **Infraestrutura:** Servidor Windows para autenticação e gestão de acessos, serviço de Área de Trabalho Remota (RDP) habilitado e estações de trabalho de usuários.
* **Incidente inicial:** A equipe de TI identificou um volume incomum de falhas de autenticação com origens externas, levantando suspeita de ataque de força bruta contra as credenciais corporativas.

---

## 📁 Estrutura do Repositório

```text
brute-force-detection-incident-response/
├── README.md
├── 01-coleta-de-evidencias/
│   ├── logs-autenticacao.csv           # Base de logs formato padrão (RFC 4180)
│   └── logs-autenticacao-excel.csv     # Base formatada para visualização direta no Excel (pt-BR)
├── 02-investigacao-de-logs/
│   ├── analise-de-eventos.md           # Investigação técnica, correlação temporal e métricas
│   └── identificacao-de-anomalias.md   # Mapeamento do padrão de ataque e comprometimento
├── 03-deteccao-de-ameacas/
│   ├── regras-de-deteccao.md           # Regras SIEM/Sigma e alertas recomendados
│   └── indicadores-suspeitos.md        # Indicadores de Comprometimento (IoCs)
├── 04-resposta-a-incidentes/
│   ├── plano-de-resposta.md            # Fases de Resposta a Incidentes (NIST SP 800-61)
│   └── medidas-de-contencao.md         # Ações imediatas de erradicação e recuperação
├── 05-gestao-de-riscos/
│   └── matriz-de-riscos.md             # Avaliação de riscos e mapeamento de controles ISO/IEC 27001
├── 06-relatorios/
│   ├── relatorio-tecnico.md            # Documentação técnica aprofundada para o time de TI/Segurança
│   └── resumo-executivo.md             # Visão estratégica e impacto para a diretoria/C-Level
└── evidencias/
    └── capturas-de-tela/               # Registros visuais das análises
```

---

## 🧭 Fases do Estudo de Caso

- [x] **Fase 1:** Definição do cenário, contextualização e coleta de evidências.
- [x] **Fase 2:** Investigação de logs, correlação de eventos (4624 / 4625) e identificação de anomalias.
- [x] **Fase 3:** Detecção de ameaças, regras de SIEM e mapeamento de IoCs.
- [x] **Fase 4:** Plano de resposta a incidentes e medidas de contenção (NIST SP 800-61).
- [x] **Fase 5:** Gestão de riscos, controles ISO/IEC 27001 e parecer DPO (LGPD: avaliação de ransomware/criptomoedas, dano reputacional e prevenção a fraudes/golpes governamentais contra clientes).
- [x] **Fase 6:** Elaboração de Relatório Técnico forense e Resumo Executivo para liderança (justificativa de investimento em VPN + MFA e ROI de segurança).
