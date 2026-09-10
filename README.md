# 🧪 HESK — Complete Software Quality Assurance Project

## 📌 Sobre el proyecto

Este repositorio documenta un **proyecto integral de Calidad de Software (QA)** realizado sobre **HESK**, una aplicación web de Help Desk y gestión de tickets.

🔗 Aplicación utilizada: [HESK Help Desk](https://www.hesk.com/)

El objetivo del proyecto es simular un **proceso de aseguramiento de calidad cercano a un entorno profesional**, cubriendo diferentes etapas del ciclo de testing: desde el entendimiento inicial del producto y la definición de la estrategia de pruebas, hasta la automatización y ejecución de pruebas dentro de un flujo de Integración Continua.

El repositorio funcionará como evidencia práctica de conocimientos en:

* QA Manual
* QA Funcional
* QA Analyst
* API Testing
* Database Testing
* Test Automation
* Continuous Integration

---

# 🎯 Objetivo

Desarrollar un proceso completo de **Software Quality Assurance** sobre HESK, aplicando técnicas, herramientas y buenas prácticas utilizadas en proyectos reales.

Durante el proyecto se trabajará sobre distintas capas de la aplicación:

**UI → API → Base de Datos → Automatización → CI/CD**

El objetivo no será únicamente ejecutar pruebas, sino también documentar todo el proceso de calidad:

* Análisis del producto
* Identificación de riesgos
* Estrategia de pruebas
* Diseño de escenarios
* Ejecución de pruebas
* Gestión de defectos
* Validación de APIs
* Validación de datos
* Automatización
* Reportes
* Integración Continua

---

# 🖥️ Sistema bajo pruebas

## HESK Help Desk

HESK es un sistema de **Help Desk / Ticket Management** que permite gestionar solicitudes de soporte mediante tickets.

Entre los principales flujos que serán analizados se encuentran:

* Registro y gestión de tickets.
* Acceso de clientes.
* Gestión de usuarios.
* Gestión de agentes.
* Categorías de soporte.
* Estados y prioridades.
* Respuestas a tickets.
* Archivos adjuntos.
* Gestión administrativa.
* Búsqueda y filtrado.
* Configuraciones del sistema.

Estos flujos servirán como base para diseñar escenarios de prueba funcionales, exploratorios, de integración y automatizados.

---

# 🔄 Flujo general del proyecto

```text
Discovery
   ↓
Test Strategy
   ↓
Manual Testing
   ↓
Database Testing
   ↓
API Testing
   ↓
API Automation
   ↓
UI Automation
   ↓
Continuous Integration
```

Cada etapa contará con su propia documentación, evidencias, resultados y conclusiones.

---

# 📂 Estructura del proyecto

```text
Hesk-Project/
│
├── 00-discovery/
│
├── 01-test-strategy/
│
├── 02-manual-testing/
│
├── 03-database-testing/
│
├── 04-api-testing/
│
├── 05-api-automation/
│
├── 06-ui-automation/
│
├── 07-ci-cd/
│
├── docs/
│
└── README.md
```

---

# 🔎 00 — Discovery

La primera etapa estará enfocada en conocer la aplicación antes de comenzar formalmente con las pruebas.

Se realizará un análisis exploratorio inicial para comprender:

* Objetivo del sistema.
* Usuarios principales.
* Roles disponibles.
* Módulos.
* Funcionalidades.
* Flujos principales.
* Dependencias.
* Reglas de negocio.
* Posibles riesgos.

### Entregables

* Mapa funcional de la aplicación.
* Identificación de módulos.
* Identificación de usuarios y roles.
* Flujos principales.
* Primer análisis de riesgos.
* Notas de exploración inicial.

---

# 📋 01 — Test Strategy

Luego del descubrimiento inicial se desarrollará la **Estrategia de Pruebas**.

Esta etapa establecerá cómo se abordará la calidad del sistema.

Se definirán:

* Objetivos de calidad.
* Alcance.
* Fuera de alcance.
* Tipos de prueba.
* Niveles de prueba.
* Técnicas de diseño.
* Priorización basada en riesgos.
* Ambientes.
* Datos de prueba.
* Herramientas.
* Criterios de entrada.
* Criterios de salida.
* Gestión de defectos.
* Evidencias.
* Métricas.

La estrategia servirá como guía para el resto del proyecto.

---

# 🧪 02 — Manual Testing

En esta etapa se realizarán pruebas funcionales y exploratorias sobre los principales módulos de HESK.

### Técnicas

Se aplicarán diferentes técnicas de diseño de pruebas, entre ellas:

* Partición de equivalencia.
* Valores límite.
* Tablas de decisión.
* Transición de estados.
* Error Guessing.
* Pruebas basadas en escenarios.
* Exploratory Testing mediante Charters.

### Tipos de pruebas

* Smoke Testing
* Functional Testing
* Exploratory Testing
* Negative Testing
* Regression Testing
* End-to-End Testing

### Entregables

* Escenarios de prueba.
* Casos de prueba.
* Checklists.
* Exploratory Testing Charters.
* Evidencias.
* Resultados de ejecución.
* Reportes de defectos.
* Matriz de trazabilidad.

---

# 🐞 Gestión de defectos

Los defectos encontrados durante las pruebas serán documentados siguiendo una estructura profesional.

Cada bug podrá contener:

* ID.
* Título.
* Descripción.
* Ambiente.
* Precondiciones.
* Pasos para reproducir.
* Resultado esperado.
* Resultado actual.
* Severidad.
* Prioridad.
* Evidencia.
* Estado.

La gestión y organización del proyecto podrá complementarse mediante **Jira**.

---

# 🗄️ 03 — Database Testing

También se realizarán validaciones sobre la capa de datos del sistema.

El objetivo será comprobar que las operaciones realizadas desde la aplicación se reflejen correctamente en la base de datos.

Se trabajará con consultas SQL para validar:

* Inserción de registros.
* Actualización de información.
* Eliminaciones.
* Relaciones entre tablas.
* Integridad de datos.
* Valores nulos.
* Duplicados.
* Consistencia.
* Reglas de negocio.

### SQL

Se utilizarán consultas como:

```sql
SELECT
INSERT
UPDATE
DELETE
JOIN
GROUP BY
ORDER BY
WHERE
HAVING
```

También se analizarán relaciones y estructuras de las tablas relevantes.

---

# 🔌 04 — API Testing

Las APIs disponibles o utilizadas dentro del ecosistema del proyecto serán analizadas mediante pruebas de integración.

Las pruebas contemplarán:

* Métodos HTTP.
* Headers.
* Query Parameters.
* Path Parameters.
* Request Body.
* Authentication.
* Status Codes.
* Response Body.
* Response Headers.
* Response Time.
* Validaciones positivas.
* Validaciones negativas.

### Herramientas

Principalmente:

* Postman
* JSON
* JSON Schema

### Validaciones

Ejemplo:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

También se podrán validar:

* Tipos de datos.
* Campos obligatorios.
* Estructura JSON.
* Reglas de negocio.
* Manejo de errores.

---

# 🤖 05 — API Automation

Los escenarios API de mayor valor serán trasladados posteriormente a pruebas automatizadas.

Dependiendo del alcance del proyecto podrán utilizarse herramientas como:

* Java
* Rest Assured
* JUnit / TestNG
* Maven
* JSON Schema Validation
* Allure Report

Ejemplo conceptual:

```java
given()
    .baseUri(BASE_URL)
.when()
    .get("/endpoint")
.then()
    .statusCode(200);
```

El objetivo será construir una suite mantenible de pruebas API automatizadas.

---

# 🎭 06 — UI Automation

Los escenarios funcionales más críticos también serán candidatos para automatización de interfaz.

La automatización estará enfocada principalmente en:

* Smoke Tests.
* Regression Tests.
* Flujos críticos.
* Casos repetitivos.
* Validaciones End-to-End.

Las tecnologías consideradas incluyen:

* Playwright
* TypeScript
* Selenium WebDriver
* Java

Se aplicarán conceptos como:

* Page Object Model.
* Localizadores mantenibles.
* Esperas.
* Fixtures.
* Datos de prueba.
* Assertions.
* Reutilización de código.
* Reportes de ejecución.

---

# ⚙️ 07 — Continuous Integration

La etapa final del proyecto buscará integrar las pruebas automatizadas dentro de un pipeline de **Continuous Integration**.

Se utilizará principalmente:

### GitHub Actions

El objetivo será ejecutar automáticamente las suites de prueba ante determinados eventos del repositorio.

Por ejemplo:

```text
Push / Pull Request
        ↓
GitHub Actions
        ↓
Instalación de dependencias
        ↓
API Tests
        ↓
UI Tests
        ↓
Generación de resultados
```

Esto permitirá detectar regresiones de manera temprana y mantener feedback constante sobre la calidad del sistema.

---

# 📊 Reportes y métricas

Durante el proyecto se podrán registrar métricas como:

* Casos de prueba diseñados.
* Casos ejecutados.
* Casos Passed.
* Casos Failed.
* Casos Blocked.
* Bugs encontrados.
* Bugs por severidad.
* Cobertura funcional.
* Porcentaje de automatización.
* Resultados de regresión.

Ejemplo:

| Métrica         | Resultado   |
| --------------- | ----------- |
| Test Cases      | En progreso |
| Passed          | —           |
| Failed          | —           |
| Bugs            | —           |
| Automated Tests | —           |
| API Tests       | —           |

Los valores serán actualizados conforme avance el proyecto.

---

# 🛠️ Herramientas

A lo largo del proyecto se utilizarán diferentes herramientas del ecosistema QA.

| Área             | Herramientas                       |
| ---------------- | ---------------------------------- |
| Gestión          | Jira                               |
| Documentación    | GitHub / Markdown                  |
| Versionado       | Git / GitHub                       |
| Manual Testing   | Jira / Checklists / Test Cases     |
| API Testing      | Postman                            |
| Database Testing | SQL                                |
| API Automation   | Java + Rest Assured                |
| UI Automation    | Playwright + TypeScript / Selenium |
| Reporting        | Allure / Playwright Report         |
| CI/CD            | GitHub Actions                     |

---

# 🧠 Competencias aplicadas

Este proyecto busca demostrar conocimientos prácticos relacionados con:

**QA Manual**

`Test Cases` `Exploratory Testing` `Bug Reporting` `Regression Testing` `E2E Testing`

**QA Analyst**

`Test Strategy` `Risk Analysis` `Traceability` `Requirements Analysis` `Test Planning`

**API Testing**

`Postman` `REST API` `JSON` `HTTP` `JSON Schema`

**Database Testing**

`SQL` `Data Validation` `Joins` `Database Testing`

**Automation**

`Playwright` `TypeScript` `Java` `Rest Assured` `Selenium`

**DevOps / CI**

`Git` `GitHub` `GitHub Actions` `CI/CD`

---

# 📈 Estado del proyecto

🚧 **Proyecto actualmente en desarrollo**

El repositorio será actualizado progresivamente a medida que se complete cada etapa.

```text
Discovery                🔄
Test Strategy            ⏳
Manual Testing           ⏳
Database Testing         ⏳
API Testing              ⏳
API Automation           ⏳
UI Automation            ⏳
Continuous Integration   ⏳
```

Los estados se actualizarán conforme avance el proyecto.

---

# 🎯 Resultado esperado

Al finalizar el proyecto, este repositorio contendrá evidencia de un proceso completo de Calidad de Software:

```text
Análisis del producto
        ↓
Estrategia de pruebas
        ↓
Diseño de pruebas
        ↓
Pruebas manuales
        ↓
Gestión de defectos
        ↓
Validaciones SQL
        ↓
API Testing
        ↓
API Automation
        ↓
UI Automation
        ↓
Continuous Integration
        ↓
Reportes de calidad
```

El objetivo es demostrar no solamente conocimiento sobre herramientas de testing, sino también la capacidad de **analizar un producto, diseñar una estrategia de calidad, ejecutar pruebas, identificar riesgos, documentar resultados y construir automatizaciones mantenibles**.

---

# 👨‍💻 Autor

**Joaquin Frias**

Técnico Superior en Desarrollo de Software
QA / Software Tester

Especialización en:

`QA Manual` · `API Testing` · `SQL` · `Test Automation` · `Playwright` · `TypeScript`

GitHub: [joaquinfriass](https://github.com/joaquinfriass)

---

> Este proyecto forma parte de mi portfolio profesional de **Software Quality Assurance**, donde documento de forma práctica el proceso completo de calidad aplicado sobre aplicaciones reales.
