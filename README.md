# 📦 Caso de Estudio — YoLoEnvío

> **Nota de confidencialidad:** El código fuente de este proyecto no es público por acuerdo con el cliente. Este documento describe exclusivamente la arquitectura, decisiones técnicas e impacto medible del sistema.

---

## 🏛️ Contexto

| Campo | Detalle |
|---|---|
| **Proyecto** | YoLoEnvío — Plataforma de logística y venta de guías de envío |
| **Mi rol** | Desarrollador Full Stack (mejoras e integración de nuevos módulos sobre plataforma existente) |
| **Período** | Febrero 2023 – Abril 2026 |
| **Escala** | +279,000 usuarios registrados |

---

## 🎯 Problema

YoLoEnvío era una plataforma existente de intermediación entre usuarios finales y múltiples paqueterías. Al incorporarme al proyecto, la plataforma enfrentaba varios problemas que requerían solución:

- **Integración de paqueterías sin arquitectura desacoplada:** agregar un nuevo proveedor implicaba modificar lógica de negocio core y arriesgar integraciones existentes
- **Fraude en cuentas:** usuarios maliciosos creando cuentas masivas para abusar de precios o créditos
- **Cálculo manual de costos:** el equipo operativo revisaba a mano sobrepesos, márgenes y descuentos por cliente
- **Nula visibilidad de datos:** sin capacidad de segmentar usuarios ni analizar comportamiento para campañas de marketing
- **Cero cobertura de pruebas** en módulos transaccionales críticos
- **Sin API pública:** los desarrolladores externos no podían integrar YoLoEnvío en sus propias plataformas

---

## ✅ Mejoras Implementadas

Trabajé sobre la plataforma existente desarrollando e integrando los siguientes módulos:

- **API pública para desarrolladores externos**, permitiendo que terceros integren los servicios de envío de YoLoEnvío en sus propias aplicaciones
- **Arquitectura en capas para paqueterías**, desacoplando la lógica de integración para que agregar nuevos proveedores no afecte el sistema existente
- **Sistema de detección de fraude** por fingerprinting de dispositivo y comportamiento
- **Pipeline de datos en SQL** para sincronización con CRM y análisis en BigQuery
- **Motor de reglas de negocio** para cálculo automático de costos de guías (sobrepesos, márgenes, descuentos)
- **Plugin para Shopify** en React.js e integración con WooCommerce para distribución de guías en e-commerce
- **Suite de +30 pruebas automatizadas** sobre módulos transaccionales críticos
- **Pasarelas de pago** y autenticación con Google OAuth
- **Análisis predictivo de ventas** con BigQuery ML

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTES / CANALES                   │
│  Web App (Laravel + Blade) · Plugin Shopify · Externos  │
└───────────────────────────┬─────────────────────────────┘
                            │ HTTPS
┌───────────────────────────▼─────────────────────────────┐
│                 BACKEND — Laravel                       │
│     Auth · Rate Limiting · Fraud Detection Layer        │
│                   API Pública REST                      │
└──────┬──────────────┬───────────────┬────────────────────┘
       │              │               │
┌──────▼───┐   ┌──────▼──────┐  ┌────▼───────────────────┐
│ Módulo   │   │  Módulo     │  │  Capa de Paqueterías   │
│ Pagos    │   │  Usuarios   │  │  (Adapter Pattern)     │
│          │   │  OAuth      │  │  Proveedor A · B · …   │
└──────────┘   └──────┬──────┘  └────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│               BASE DE DATOS — MySQL                     │
│    Transacciones · Usuarios · Guías · Auditoría         │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│            PIPELINE DE DATOS — Google Cloud             │
│    BigQuery · Tablas espejo · Jobs programados          │
│    → Exportación a CRM para campañas de marketing       │
└─────────────────────────────────────────────────────────┘
```

---

## ⚙️ Decisiones de Arquitectura

**Adapter Pattern para paqueterías**
Cada proveedor de paquetería implementa una interfaz común. El núcleo del sistema no conoce los detalles de ningún proveedor específico. Agregar una nueva paquetería equivale a crear un adaptador sin tocar la lógica existente, esto resolvió el problema de acoplamiento que había en la plataforma original.

**API pública para desarrolladores**
Se diseñó y construyó una API REST que permite a desarrolladores externos integrar los servicios de envío de YoLoEnvío en sus propias aplicaciones, abriendo un nuevo canal de distribución para la plataforma.

**Fingerprinting para detección de fraude**
En lugar de validar solo por email o IP, el sistema construye un fingerprint combinando señales de dispositivo, browser y comportamiento del usuario. Los registros sospechosos se bloquean antes de completar la creación de cuenta.

**Pipeline SQL → BigQuery → CRM**
Los datos operativos de MySQL se sincronizan periódicamente a BigQuery con tablas espejo y transformaciones programadas. Desde BigQuery se generan los segmentos exportados al CRM para campañas de marketing automatizadas basadas en datos reales.

**Motor de reglas de negocio**
El cálculo de costos fue extraído a un motor de reglas configurable, eliminando la revisión manual del equipo operativo y reduciendo errores en la facturación.

---

## 🛠️ Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Backend / API | Laravel (PHP) |
| Frontend web | Laravel + Blade |
| Plugin e-commerce | React.js (Shopify), WooCommerce |
| Base de datos | MySQL |
| Data warehouse | BigQuery (Google Cloud) |
| CRM | HubSpot |
| Pagos | Conekta, PayPal |
| Autenticación | Google OAuth |
| Infraestructura | AWS, Docker |
| Testing | PHPUnit (+30 unit tests) |
| Predicción de ventas | BigQuery ML |

---

## 📊 Impacto

- **+279,000 usuarios registrados** en la plataforma
- **+2,306 cuentas fraudulentas** identificadas y bloqueadas por el sistema de detección
- **+20 GB de base de datos** estructurada y limpiada para habilitar el pipeline de datos
- **Eliminación de revisión manual** en el cálculo de costos de guías para el equipo operativo
- **+30 tests automatizados** sobre módulos transaccionales críticos con reportes para QA
- **API pública** habilitando integración de terceros con los servicios de la plataforma
- **Análisis predictivo de ventas** habilitado mediante BigQuery ML

---

## 🔐 Nota de Confidencialidad

Este proyecto fue desarrollado para un cliente privado bajo acuerdo de confidencialidad. El código fuente, las credenciales, los datos de usuarios, los nombres de proveedores y los detalles internos del negocio no son públicos. Este caso de estudio documenta únicamente arquitectura, decisiones técnicas e impacto, sin comprometer la seguridad ni la privacidad del cliente.

---

## 👤 Autor

**Alexis García Ruiz** — Desarrollador Full Stack

[![LinkedIn](https://img.shields.io/badge/LinkedIn-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/alexis-gr/)
[![GitHub](https://img.shields.io/badge/GitHub-AlexisGR--isc-black?style=flat&logo=github)](https://github.com/AlexisGR-isc)
[![Email](https://img.shields.io/badge/Email-isc.alexisgr@gmail.com-red?style=flat&logo=gmail)](mailto:isc.alexisgr@gmail.com)
