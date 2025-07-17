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



------------------------------------------------------------------------------------------------------------------------------------------



# 🛠️ Documentación - Pipeline Microservicio Gitlab

Este pipeline está diseñado para manejar el ciclo de vida de un microservicio en tres ambientes: `desarrollo`, `calidad` y `producción`. Se ejecuta automáticamente según el tipo de evento (manual, `pull_request`, etc.) y los valores de `inputs`.

---

## 🧪 Desarrollo

### ¿Cómo se ejecuta?
Se activa manualmente mediante `workflow_call` y se debe pasar:
- `branch_or_tag`: nombre del branch (`develop`, `feature/*`, etc.).
- `environment`: debe ser igual a `desarrollo`.

### ¿Qué hace?

#### Jobs ejecutados:
- `set-environment`: Determina el entorno según el branch.
- `build`: Compila el microservicio usando Maven.
- `deploy-artifact-maven`: Publica el artefacto en Artifactory (si los jobs de calidad se activan).
- `build-image-to-delivery`: Construye la imagen Docker y la publica en el registro interno.
- `prisma-cloud-scan`: Realiza un escaneo de seguridad de la imagen Docker (Prisma Cloud).
- `delivery-app`: Despliega el microservicio en OCP.
- `burp-scan`: Ejecuta análisis de seguridad dinámico (Burp Suite).

> ⚠️ Los jobs de calidad (`snyk_SCA`, `snyk_SAST`, `sonarqube-unit-test`) están definidos pero actualmente deshabilitados (`if: false`).

---

## ✅ Calidad

### ¿Cómo se ejecuta?
Automáticamente cuando se crea un Pull Request cuya rama destino es `calidad`.

### ¿Qué hace?

#### Jobs ejecutados:
- `set-environment`: Detecta que se está trabajando sobre calidad.
- `build`: Compila el microservicio.
- `pre-release`: Ejecuta una pre-liberación del artefacto.
- `deploy-artifact-maven`: Publica el artefacto en Artifactory.
- `build-image-to-delivery`: Crea y publica la imagen Docker.
- `delivery-app`: Despliega la app en el entorno de OCP correspondiente.

> ℹ️ Los escaneos de seguridad y el Burp scan **no se ejecutan** en este ambiente.

---

## 🚀 Producción

### ¿Cómo se ejecuta?
Se activa manualmente, pasando:
- `branch_or_tag`: el tag que representa la versión a liberar.
- `environment`: debe ser igual a `produccion`.

### ¿Qué hace?

#### Jobs ejecutados:
- `set-environment`: Detecta que se trata de un entorno productivo.
- `release`: Ejecuta el proceso de liberación oficial (tag) del microservicio.
- `deploy-artifact-maven`: Publica el artefacto si los jobs previos se activaran.
- `build-image-to-delivery`: Construye y publica la imagen para producción.
- `delivery-app`: Despliega la aplicación a OCP.

---

## 📋 Jobs por Ambiente

| Job                         | Desarrollo | Calidad | Producción |
|----------------------------|------------|---------|------------|
| set-environment            | ✅         | ✅      | ✅         |
| build                      | ✅         | ✅      | ❌         |
| snyk_SCA                   | ❌ (definido pero desactivado) | ❌ | ❌ |
| snyk_SAST                  | ❌ (definido pero desactivado) | ❌ | ❌ |
| sonarqube-unit-test        | ❌ (definido pero desactivado) | ❌ | ❌ |
| deploy-artifact-maven      | ✅         | ✅      | ✅         |
| build-image-to-delivery    | ✅         | ✅      | ✅         |
| pre-release                | ❌         | ✅      | ❌         |
| release                    | ❌         | ❌      | ✅         |
| prisma-cloud-scan          | ✅         | ❌      | ❌         |
| delivery-app               | ✅         | ✅      | ✅         |
| burp-scan                  | ✅         | ❌      | ❌         |

---

## 🧾 Inputs del Workflow

- `project`: Nombre del proyecto (ej: `JRVS`).
- `version`: Versión que se desea desplegar en Artifactory.
- `environment`: Ambiente de despliegue (`desarrollo`, `calidad`, `produccion`).
- `branch_or_tag`: Branch o tag sobre el que se ejecutará la acción.

---

## 🔒 Secrets requeridos

| Secret                        | Descripción                                                        |
|------------------------------|--------------------------------------------------------------------|
| `GH_PAT`                     | Token de acceso a GitHub.                                         |
| `ARTIFACTORY_USER`           | Usuario para Artifactory.                                         |
| `ARTIFACTORY_PASSWORD`       | Contraseña para Artifactory.                                      |
| `SNYK_TOKEN`                 | Token de autenticación de Snyk (aunque deshabilitado).             |
| `SONAR_TOKEN`                | Token para SonarQube (aunque deshabilitado).                      |
| `EXTERNAL_REGISTRY_USR`      | Usuario para acceso al registry externo (Docker/Openshift).        |
| `EXTERNAL_REGISTRY_PSW`      | Contraseña del registry externo.                                  |
| `OCP_TOKEN`                  | Token para autenticación contra Openshift.                         |
| `OC_TOKEN`                   | Token adicional para OCP (usado en `delivery-app`).               |
| `PRISMA_URL`, `PRISMA_USER`, `PRISMA_PASS` | Credenciales para escaneo Prisma Cloud.                     |
| `BURPSUITE_API_KEY`          | API Key para ejecutar escaneo dinámico con Burp Suite.             |

---

## 🧠 Notas adicionales

- El job `set-environment` determina el ambiente a partir del branch o base del PR.
- Los análisis de calidad con Snyk y Sonar están definidos pero deshabilitados.
- `prisma-cloud-scan` y `burp-scan` solo se ejecutan en el ambiente `desarrollo`.
- `delivery-app` siempre corre al final si no hubo fallas previas.
- Los despliegues y builds usan workers `self-hosted` para ejecutar sobre infraestructura interna.



