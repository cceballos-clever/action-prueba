
# Manual de Migración de Librerías a GitHub Package

## Librerías Migradas

Se migraron las siguientes librerías a GitHub Package:

- [ng-microkernel](https://github.com/orgs/pichincha-peru/packages/npm/package/ng-microkernel)
- [tailwind-config](https://github.com/orgs/pichincha-peru/packages/npm/package/tailwind-config)

Los cambios se realizaron en el branch [`feature/github-package`](https://github.com/pichincha-peru/frt-xdig-libraries/tree/feature/github-package) del repositorio:

- https://github.com/pichincha-peru/frt-xdig-libraries

## Publicación de las Librerías

Para hacer el push de las librerías a GitHub Package:

1. Usar el script [`publish-library-pichincha.sh`](https://github.com/pichincha-peru/frt-xdig-libraries/blob/feature/github-package/publish-library-pichincha.sh)
2. Agregar el Personal Access Token (PAT) donde se solicita en el script.

## Descarga de las Librerías

Para descargar las librerías es necesario configurar el archivo `.npmrc` así:

```ini
@pichincha-peru:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=$TOKEN
```

> Reemplazar `$TOKEN` por el Personal Access Token correspondiente.

## Pruebas de Instalación

Se utilizó el repositorio [`frt-xdig-mf-loan`](https://github.com/pichincha-peru/frt-xdig-mf-loan/tree/feature/github-package) en la rama `feature/github-package` para realizar pruebas de instalación y descarga.

---

## Pasos para Migrar Librerías

1. **Modificar `.npmrc`**: agregar el token como se indica más arriba.
2. **Cambiar `package.json`**: actualizar las referencias a las librerías para que apunten a `@pichincha-peru/...`.
3. **Eliminar `package-lock.json`**: para evitar conflictos con la resolución de paquetes.
4. **Instalación desde GitHub Package**:

   Ver instrucciones de instalación directamente en la [página del paquete](https://github.com/orgs/pichincha-peru/packages/npm/package/tailwind-config).

5. **Actualizar Imports**:

   Cambiar en todos los archivos donde se importaban las librerías así:

   **Antes**:

   ```ts
   import { MicroFrontendsModule } from '@xdig/ng-microkernel/micro-frontend';
   ```

   **Después**:

   ```ts
   import { MicroFrontendsModule } from '@pichincha-peru/ng-microkernel/micro-frontend';
   ```

---

## Notas

- Esta configuración es necesaria tanto para ambientes de desarrollo como para CI/CD.
- La instalación puede requerir autenticación en cada entorno donde se utilice.

---

## Contacto

Ante cualquier duda o inconveniente, contactar con el equipo responsable del mantenimiento de estas librerías.
