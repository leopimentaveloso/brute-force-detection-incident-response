# Detección de Fuerza Bruta y Respuesta a Incidentes

Proyecto práctico de ciberseguridad defensiva (*Blue Team*): investigación forense de registros de autenticación de Windows, mitigación de un ataque de fuerza bruta remoto y estructuración de un plan de respuesta a incidentes alineado con **ISO/IEC 27001:2022** y **NIST SP 800-61 Rev. 3**.

> **Nota sobre las direcciones IP utilizadas:** Los bloques `192.0.2.0/24` y `203.0.113.0/24` están reservados para documentación y pruebas según la **RFC 5737**. En este laboratorio, `192.0.2.0/24` representa la red corporativa interna ficticia y `203.0.113.50` representa un origen externo no autorizado ficticio. No corresponden a redes ni máquinas reales.

---

## 🌐 Versiones de Idioma
[🇧🇷 Português (Original)](README.md) | [🇺🇸 English](README.en.md) | [🇪🇸 Español](README.es.md)

---

## 🎯 Objetivo del Proyecto

Detectar, investigar, contener y documentar un ataque automatizado de fuerza bruta (*Brute Force / Password Spraying*) contra servicios de Escritorio Remoto (RDP), evaluar el compromiso de credenciales en cuentas de servicio y definir planes de acción técnicos y ejecutivos.

Este proyecto complementa los laboratorios de seguridad ofensiva (como fuerza bruta con *Medusa*), aportando la perspectiva empresarial **defensiva** (SOC / Respuesta a Incidentes / GRC).

---

## 🏢 Caso de Estudio Ficticio: TechSolutions Ltda.

* **Organización:** TechSolutions Ltda.
* **Sector:** Servicios de TI y soporte empresarial.
* **Tamaño:** ~50 colaboradores.
* **Infraestructura:** Servidor Windows para autenticación de dominio, servicios de Escritorio Remoto (RDP) expuestos a internet, estaciones de trabajo.
* **Evento Detonante:** Alertas automáticas identificaron un aumento anómalo de intentos fallidos de inicio de sesión procedentes de una dirección IP externa, dirigidos contra cuentas corporativas clave.

---

## 📁 Estructura del Repositorio

```text
brute-force-detection-incident-response/
├── README.md                           # Documentación principal (pt-BR)
├── README.en.md                        # Documentación en inglés
├── README.es.md                        # Documentación en español
├── 01-coleta-de-evidencias/
│   ├── logs-autenticacao.csv           # Registros de autenticación (RFC 4180)
│   └── logs-autenticacao-excel.csv     # Formato para Microsoft Excel (pt-BR/Latin-1)
├── 02-investigacao-de-logs/
│   ├── analise-de-eventos.md           # Métricas, análisis volumétrico y cronología
│   └── identificacao-de-anomalias.md   # Patrón de ataque y anomalías forenses
├── 03-deteccao-de-ameacas/
│   ├── regras-de-deteccao.md           # Correlación SIEM, regla Sigma y hardening
│   └── indicadores-suspeitos.md        # Indicadores de Compromiso (IoCs) y MITRE ATT&CK
├── 04-resposta-a-incidentes/
│   ├── plano-de-resposta.md            # Ciclo de Vida de Incidentes NIST SP 800-61
│   └── medidas-de-contencao.md         # Acciones inmediatas de contención (15 min)
├── 05-gestao-de-riscos/
│   └── matriz-de-riscos.md             # Matriz de Riesgo 5x5, ISO 27001:2022 y DPO
├── 06-relatorios/
│   ├── relatorio-tecnico.md            # Informe forense detallado para TI/SOC
│   └── resumo-executivo.md             # Informe estratégico de alto nivel para C-Level
└── evidencias/
    └── capturas-de-tela/               # Capturas visuales y evidencias
```

---

## 🧭 Hitos Completados del Proyecto

- [x] **Fase 1: Recopilación de Evidencias:** Ingesta de 24 eventos forenses de autenticación (Event IDs 4624 y 4625).
- [x] **Fase 2: Investigación Forense de Logs:** Detección de ataques automatizados de alta velocidad y compromiso de la cuenta `svc_backup` a las 09:18:04.
- [x] **Fase 3: Detección de Amenazas e Ingeniería SIEM:** Creación de lógica de correlación, políticas de bloqueo y una **Regla Sigma (YAML)** estándar abierta mapeada a MITRE ATT&CK (T1110, T1133).
- [x] **Fase 4: Respuesta a Incidentes y Contención (NIST SP 800-61 Rev. 3):** Protocolo de 15 minutos para cierre de sesión, bloqueo en firewall perimetral, rotación de credenciales y aislamiento del host.
- [x] **Fase 5: GRC, ISO/IEC 27001:2022 y Evaluación DPO:** Matriz de Riesgo 5x5, mapeo de controles del Anexo A (A.5.15, A.8.2, A.8.5, A.8.20, A.5.24) y notificación regulatoria de privacidad (LGPD / RGPD), abordando riesgos de ransomware/extorsión en criptomonedas, daño reputacional y prevención de fraudes/cobros falsos gubernamentales contra clientes.
- [x] **Fase 6: Informes:** Entrega de informe técnico forense para ingeniería y resumen ejecutivo presentando el ROI de seguridad, justificación de inversión en VPN + MFA y marco de decisión para la dirección.
