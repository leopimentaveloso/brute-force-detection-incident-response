# Brute Force Detection & Incident Response

A hands-on Blue Team cybersecurity and digital forensics project: investigating Windows authentication logs, mitigating a remote brute-force attack, and structuring an incident response framework aligned with **ISO/IEC 27001** and **NIST SP 800-61 Rev. 2**.

---

## 🌐 Language Versions
[🇧🇷 Português (Original)](README.md) | [🇺🇸 English](README.en.md) | [🇪🇸 Español](README.es.md)

---

## 🎯 Project Objective

To detect, investigate, contain, and document an automated brute-force / password spraying attack against Remote Desktop Protocol (RDP) services, evaluate credential compromise on service accounts, and establish technical and executive response plans.

This project directly complements offensive security labs (such as brute-force with *Medusa*), delivering the enterprise **defensive perspective** (SOC / Incident Response / GRC).

---

## 🏢 Fictional Case Study: TechSolutions Ltd.

* **Organization:** TechSolutions Ltd.
* **Industry:** IT services and corporate enterprise support.
* **Size:** ~50 employees.
* **Infrastructure:** Windows Server for domain authentication, Remote Desktop Services (RDP) exposed externally, employee workstations.
* **Trigger Event:** Automated alerts identified an abnormal spike in failed login attempts originating from an external IP address, targeting key corporate user accounts.

---

## 📁 Repository Structure

```text
brute-force-detection-incident-response/
├── README.md                           # Main documentation (pt-BR)
├── README.en.md                        # English documentation
├── README.es.md                        # Spanish documentation
├── 01-coleta-de-evidencias/
│   ├── logs-autenticacao.csv           # Authentication logs (Standard RFC 4180)
│   └── logs-autenticacao-excel.csv     # Formatted for Microsoft Excel (pt-BR/Latin-1)
├── 02-investigacao-de-logs/
│   ├── analise-de-eventos.md           # Metrics, volumetric analysis, and timeline
│   └── identificacao-de-anomalias.md   # Attack pattern and forensic anomalies
├── 03-deteccao-de-ameacas/
│   ├── regras-de-deteccao.md           # SIEM correlation, Sigma rule, and hardening
│   └── indicadores-suspeitos.md        # Indicators of Compromise (IoCs) & MITRE ATT&CK
├── 04-resposta-a-incidentes/
│   ├── plano-de-resposta.md            # NIST SP 800-61 Incident Response Lifecycle
│   └── medidas-de-contencao.md         # Immediate 15-minute containment actions
├── 05-gestao-de-riscos/
│   └── matriz-de-riscos.md             # 5x5 Risk Matrix, ISO 27001:2022 & Privacy/DPO
├── 06-relatorios/
│   ├── relatorio-tecnico.md            # Deep-dive forensic report for IT/SOC
│   └── resumo-executivo.md             # High-level strategic report for C-Level & Board
└── evidencias/
    └── capturas-de-tela/               # Visual records and artifacts
```

---

## 🧭 Project Roadmap & Completed Milestones

- [x] **Phase 1: Evidence Collection & Scenario Definition:** Ingestion of 24 forensic authentication events (Event IDs 4624 & 4625).
- [x] **Phase 2: Log Investigation & Forensic Analysis:** Uncovered high-velocity automated password spraying and credential compromise on `svc_backup` at 09:18:04.
- [x] **Phase 3: Threat Detection & SIEM Engineering:** Created SIEM correlation logic, account lockout policies, and an open-standard **Sigma Rule (YAML)** mapped to MITRE ATT&CK (T1110, T1133).
- [x] **Phase 4: Incident Response & Containment (NIST SP 800-61):** 15-minute protocol executing session kill, perimeter firewall block, credential rotation, and host isolation.
- [x] **Phase 5: GRC, ISO/IEC 27001:2022 & DPO Assessment:** 5x5 Risk Matrix, Annex A controls mapping (A.5.15, A.8.5, A.8.18, A.8.20), and regulatory notification requirements under data protection laws (LGPD / GDPR).
- [x] **Phase 6: Reporting:** Delivered both a technical forensic report for engineering and an executive summary with business ROI for corporate leadership.
