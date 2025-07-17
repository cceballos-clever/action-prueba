# Documentación - Microfrontend Pipeline

Este pipeline está diseñado para ejecutarse en diferentes ambientes (`desarrollo`, `calidad` y `producción`) dependiendo de los inputs o eventos de GitHub. A continuación, se detalla cómo se ejecuta cada uno y qué hace.

---

## 🧪 Desarrollo

### ¿Cómo se ejecuta?
El pipeline de desarrollo se activa al **ejecutar el workflow manualmente** y pasar el branch como `input` (por ejemplo, `develop`, `feature/...`, etc.) usando el parámetro `branch_or_tag` e indicando el entorno como `desarrollo`.

### ¿Qué hace?
Cuando el `input.environment` es igual a `desarrollo`, se ejecutan los siguientes jobs:

- `build-node`: Construye el microfrontend usando Node.js.
- `sonarqube-unit-tests`: Ejecuta análisis estático y pruebas unitarias con SonarQube.
- `snyk_SCA`: Ejecuta análisis de dependencias (SCA) con Snyk.
- `snyk_SAST`: Ejecuta análisis de seguridad de código estático (SAST) con Snyk.

---

## ✅ Calidad

### ¿Cómo se ejecuta?
El pipeline de calidad se activa automáticamente al **crear un Pull Request hacia la rama `calidad`**.

### ¿Qué hace?
Cuando el evento `pull_request` apunta a la rama `calidad`, se ejecuta:

- `pre-release`: Realiza una pre-liberación para validar el estado del microfrontend antes de pasar a producción.

---

## 🚀 Producción

### ¿Cómo se ejecuta?
El pipeline de producción se activa al **ejecutar el workflow manualmente** y pasar el `tag` como `input` (en `branch_or_tag`) e indicar el entorno como `produccion`.

### ¿Qué hace?
Cuando el `input.environment` es igual a `produccion`, se ejecuta:

- `release`: Lanza la versión indicada por el tag al ambiente productivo.
- Luego, si estuvieran habilitados, podrían ejecutarse también los despliegues:
  - `deploy-to-azure-app-service`: Despliegue hacia un Azure App Service (actualmente con `if: false`).
  - `deploy-to-azure-blob-storage-cdn`: Despliegue a Azure Blob Storage + CDN para proyectos como `xdig`.

---

## 🧾 Inputs del Workflow

- `project`: Nombre del proyecto.
- `deploy_on`: Plataforma de despliegue (`app-service`, `blob-storage`, etc.).
- `environment`: Ambiente destino (`desarrollo`, `calidad`, `produccion`).
- `branch_or_tag`: Branch o Tag a trabajar/desplegar.
- `npm-install-flags`: Flags personalizados para `npm install`.

---

## 🔒 Secrets requeridos

- `GH_PAT`: Token de acceso personal a GitHub (obligatorio).
- `AZURE_STORAGE_USERNAME`, `AZURE_STORAGE_PASSWORD`: Acceso a cuenta de Azure (opcional).
- `SNYK_TOKEN`, `SONAR_TOKEN`, `SONAR_HOST_URL`, etc.: Requeridos para herramientas externas (SonarQube, Snyk, etc.).

---

## 🧠 Notas adicionales

- El job `set-environment` determina el entorno a partir del `branch_or_tag` o de la rama base del PR.
- El job `deploy-to-azure-app-service` está deshabilitado (`if: false`), pero puede activarse para despliegues productivos a App Service.
- El job `deploy-to-azure-blob-storage-cdn` se ejecuta si el `project` es `xdig`.

