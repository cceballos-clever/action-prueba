# 📦 Documentación - Pipeline Microfrontend Gitlab

Este pipeline está diseñado para ejecutarse en diferentes ambientes (`desarrollo`, `calidad` y `producción`) dependiendo de los inputs o eventos de GitHub. A continuación, se detalla cómo se ejecuta cada uno y qué hace.

---

## 🧪 Desarrollo

### ¿Cómo se ejecuta?
El pipeline de desarrollo se activa al **ejecutar el workflow manualmente** (mediante `workflow_dispatch` o `workflow_call`) pasando el nombre de un branch (`develop`, `feature/*`, etc.) en el parámetro `branch_or_tag`, y estableciendo el `input.environment` como `desarrollo`.

### ¿Qué hace?
Cuando el `input.environment` es igual a `desarrollo`, se ejecutan los siguientes jobs:

#### Jobs ejecutados:
- `set-environment`: Determina el entorno a partir del branch.
- `build-node`: Construye el microfrontend usando Node.js.
- `sonarqube-unit-tests`: Ejecuta análisis estático y pruebas unitarias con SonarQube.
- `snyk_SCA`: Ejecuta análisis de dependencias (SCA) con Snyk.
- `snyk_SAST`: Ejecuta análisis de seguridad de código estático (SAST) con Snyk.

> 💡 Nota: El job `sonarqube` está presente pero deshabilitado (`if: false`).

---

## ✅ Calidad

### ¿Cómo se ejecuta?
El pipeline de calidad se activa automáticamente al **crear un Pull Request cuya base sea la rama `calidad`**.

### ¿Qué hace?
Cuando el evento `pull_request` apunta a la rama `calidad`, se ejecuta:

#### Jobs ejecutados:
- `set-environment`: Establece el ambiente en base al destino del PR.
- `pre-release`: Ejecuta la pre-liberación para validación previa al pase a producción.

---

## 🚀 Producción

### ¿Cómo se ejecuta?
El pipeline de producción se activa al **ejecutar el workflow manualmente** y pasar un `tag` en el parámetro `branch_or_tag`, estableciendo el `input.environment` como `produccion`.

### ¿Qué hace?
Cuando el `input.environment` es igual a `produccion`, se ejecutan:

#### Jobs ejecutados:
- `set-environment`: Establece el entorno productivo.
- `release`: Lanza la versión del microfrontend usando el tag proporcionado.

> 🔁 Luego de `release`, podrían ejecutarse despliegues (actualmente deshabilitados):

- `deploy-to-azure-app-service`: Despliegue a Azure App Service (`if: false`).
- `deploy-to-azure-blob-storage-cdn`: Se ejecuta si `project == 'xdig'`.

---

## 📂 Jobs por Ambiente

| Ambiente     | Jobs ejecutados                                                                 |
|--------------|----------------------------------------------------------------------------------|
| desarrollo   | set-environment, build-node, sonarqube-unit-tests, snyk_SCA, snyk_SAST          |
| calidad      | set-environment, pre-release                                                    |
| producción   | set-environment, release (opcional: deploy-to-azure-app-service, blob-storage)  |

---

## 🧾 Inputs del Workflow

- `project`: Nombre del proyecto (ej: `xdig`).
- `deploy_on`: Plataforma de despliegue (`app-service`, `blob-storage`, etc.).
- `environment`: Ambiente destino (`desarrollo`, `calidad`, `produccion`).
- `branch_or_tag`: Branch o Tag a trabajar/desplegar.
- `npm-install-flags`: Flags personalizados para `npm install`.

---

## 🔒 Secrets requeridos

| Secret                    | Descripción                                       |
|---------------------------|---------------------------------------------------|
| `GH_PAT`                 | Token de acceso personal a GitHub (obligatorio).  |
| `AZURE_STORAGE_USERNAME` | Usuario de cuenta de Azure (opcional).            |
| `AZURE_STORAGE_PASSWORD` | Contraseña de Azure Storage (opcional).           |
| `SNYK_TOKEN`             | Token para análisis con Snyk.                     |
| `SONAR_TOKEN`            | Token de autenticación para SonarQube.            |
| `SONAR_HOST_URL`         | URL del host de SonarQube.                        |
| `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_STORAGE_KEY` | Requeridos para despliegues en Azure. |

---

## 🧠 Notas adicionales

- El job `set-environment` determina el entorno a partir del `branch_or_tag` o del destino del Pull Request.
- El job `deploy-to-azure-app-service` actualmente está deshabilitado (`if: false`), pero se puede habilitar para despliegues en Azure App Service.
- El job `deploy-to-azure-blob-storage-cdn` solo se ejecuta si el input `project` es `xdig`.

