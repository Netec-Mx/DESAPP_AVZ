# Setup Guide General
<!-- {% raw %} -->

*ESPECIFICACIÓN TÉCNICA / REV. 2.0*

## Desarrollo de aplicaciones para ambientes Web avanzado

*DESAPP_AVZ · Preparación de ambientes para laboratorios*

*Repositorio de referencia: https://github.com/Netec-Mx/DESAPP_AVZ*

## Objetivo y alcance

Establecer la configuración que Operaciones debe instalar, comprobar y entregar antes de iniciar el curso. Los capítulos 1 a 6 se ejecutan sobre TaskFlow y el capítulo 8 sobre el proyecto final. El capítulo 7 se demuestra por el instructor; no se incorpora un laboratorio adicional ni se requieren cuentas de IA para los alumnos.

| **Decisión operativa** | **Estándar de entrega** |
|---|---|
| Plataforma | Windows 11 Pro x64, actualizado y soportado. Una instancia aislada por alumno. |
| Ejecución principal | PHP, Composer, Node.js y MySQL instalados de forma nativa. |
| Contingencia | Docker Desktop operativo con contenedores Linux, Compose e imágenes locales. No es parte de las prácticas. |
| Estado académico | Sin proyecto TaskFlow, sin proyecto final, sin bases de laboratorio ni archivos .env. |
| Responsable | Operaciones prepara y valida; instructor confirma acceso y dirige el desarrollo. |

## Requisitos de capacidad

| **Recurso** | **Mínimo** | **Recomendado** |
|---|---|---|
| CPU / vCPU | 4 núcleos lógicos | 8 |
| RAM | 8 GB | 16 GB |
| SSD libre tras instalación | 40 GiB | 60 GiB |
| Virtualización | Habilitada y funcional | Validar anidada si es VM |
| GPU | No requerida | No requerida |

> La memoria mínima obliga a limitar procesos simultáneos. El motor Docker puede permanecer detenido durante las prácticas nativas; debe poder iniciarse sin asistencia. Respetar las condiciones de licencia y soporte descritas en la sección de contingencia.

# 1. Inventario y aprovisionamiento

Instalar versiones estables de mantenimiento dentro de la rama indicada; registrar la versión exacta y conservar los instaladores aprobados. No instalar dependencias del proyecto de forma global.

| **Herramienta** | **Versión / criterio** | **Preparación** |
|---|---|---|
| PHP x64 CLI | 8.5.x | Herd o distribución independiente |
| Composer | 2.x | Mismo PHP activo |
| Node.js / npm | 24 LTS / incluido | Un único runtime visible |
| MySQL Community Server | 8.4 LTS | MSI y MySQL Configurator; cliente CLI |
| Git | Estable soportada | Git CLI; identidad de commits local |
| Visual Studio Code | Estable | CLI code y extensiones |
| Bruno o Postman | Estable; uno de los dos | Bruno preferido; uso local |
| Chrome o Edge | Estable | DevTools habilitadas |
| Docker Desktop | Estable soportada | Motor Linux; Compose v2 o v5 |
| Workbench / Herd | Opcionales | Workbench para SQL; Herd como instalador |

## Secuencia de instalación

- Actualizar Windows y habilitar virtualización. Instalar PHP/Composer y Node/npm; Herd es una opción, no un requisito. Si se utiliza Herd, seleccionar PHP 8.5 y Node 24; evitar duplicar versiones en PATH.

- Instalar Git, VS Code, navegador y cliente REST. Instalar soporte PHP (Intelephense), Blade y Tailwind CSS IntelliSense; ESLint y Prettier recomendados. No exigir cuentas Copilot. Verificar git config user.name y user.email; usar la identidad autorizada del alumno.

- Instalar MySQL mediante su MSI 8.4, configurar servicio y cliente CLI. Configurar Docker Desktop según la sección 3, desde una cuenta autorizada.

## Dependencias que no debe instalar Operaciones globalmente

Laravel 13, Blade, Eloquent, Breeze, Sanctum, PHPUnit, Vite y Tailwind CSS 4 se incorporan en las prácticas mediante Composer/npm. Inertia/Vue/React/TypeScript y Pest no son prerrequisitos del recorrido principal. Python nativo es opcional; la imagen Python se conserva como respaldo.

# 2. Configuración del ambiente nativo

## Cuenta, terminal y almacenamiento

Preparar $HOME\labs para el usuario final (PowerShell). Verificar lectura, escritura y ejecución sin elevación. No usar carpetas sincronizadas o de red para vendor, node_modules y los datos de MySQL; asegurar persistencia entre sesiones.

```text
Get-Command php, composer, node, npm.cmd, git, mysql, code.cmd
php --ini
php -m
```

Si se usa Windows PowerShell y npm.ps1 está bloqueado, usar npm.cmd o aplicar una política aprobada por TI. No deshabilitar controles globales. Verificar PATH y abrir una terminal nueva tras instalar herramientas.

## Extensiones y parámetros PHP

Requeridas: ctype, curl, dom, fileinfo, filter, hash, mbstring, openssl, pcre, PDO, pdo_mysql, pdo_sqlite, session, sqlite3, tokenizer, xml y zip. Recomendada: intl. SQLite permite inicialización del esqueleto de Laravel; no se exige sqlite3 CLI ni se utiliza SQLite como motor de pruebas del curso.

```text
file_uploads = On
upload_max_filesize = 4M
post_max_size = 8M
```

Modificar el php.ini efectivo de PHP CLI/servidor de desarrollo; reiniciar procesos PHP. La aplicación limita el archivo a 2 MiB en el laboratorio 4. post_max_size debe exceder upload_max_filesize.

## MySQL: instalación y permisos

- MySQL Configurator: servicio Windows automático (por ejemplo, MySQL84), TCP 3306, escucha local 127.0.0.1 y sin regla de entrada desde red externa. Configurar bind-address=127.0.0.1 en my.ini; deshabilitar o limitar también MySQL X (33060). Registrar servicio y ruta de datos.

- Entregar credenciales administrativas locales de la instancia aislada por canal seguro, conforme al laboratorio 2 (root por TCP). Si la política impide entregar root local, Operaciones debe acompañar ese paso o acordar un procedimiento equivalente con el instructor antes de liberar el ambiente. No usar root corporativo ni un servidor multiusuario.

```text
mysql -h 127.0.0.1 -P 3306 -u root -p
-- En el cliente MySQL, ejecutar:
SELECT VERSION(), CURRENT_USER();
SHOW GRANTS;
EXIT;
```

Debe autenticarse y devolver versión 8.4.x. La comprobación de permisos de creación de esquemas, cuentas y concesiones se realiza en el smoke test de la sección 6. No crear taskflow, taskflow_test ni el usuario taskflow con antelación. Las prácticas usarán $HOME\labs\taskflow, ambas bases MySQL separadas y PHPUnit.

# 3. Docker Desktop: contingencia

Instalar Docker Desktop estable en el ambiente del alumno, no únicamente en el host. Habilitar Virtualización en BIOS/UEFI y un backend soportado (Hyper-V o WSL 2). WSL 2 no se exige como terminal académica. En una VM/VDI, comprobar virtualización anidada, compatibilidad del hipervisor y soporte/licencia aplicable.

Antes de aprovisionar, Operaciones debe confirmar los términos de Docker Desktop para la organización y el uso educativo/comercial. No presumir gratuidad para cualquier capacitación empresarial. Si el backend, licencia o permisos no se pueden aprobar, escalar la incidencia antes de entregar.

## Validar el motor y precargar imágenes

```text
docker version
docker info --format "{{.OSType}}"
docker compose version
docker pull hello-world
docker pull mysql:8.4
docker pull php:8.5-cli
docker pull composer:2
docker pull node:24
docker pull python:3.12-slim
docker run --pull=never --rm hello-world
docker image ls
```

Esperado: servidor accesible desde la cuenta del alumno, OSType linux y Compose v2 o v5. Guardar IDs/digests de imágenes aprobadas. No habilitar un daemon Docker remoto sin autenticar ni entregar credenciales del registro.

## Imagen PHP de respaldo preparada

php:8.5-cli por sí sola no garantiza pdo_mysql ni zip. En una carpeta operativa independiente, guardar este Dockerfile y construirlo antes del curso. La receta se debe validar en el ambiente final.

```text
FROM composer:2 AS composer_bin
FROM php:8.5-cli
RUN apt-get update && apt-get install -y git unzip libzip-dev \
    && docker-php-ext-install pdo_mysql zip \
    && rm -rf /var/lib/apt/lists/*
COPY --from=composer_bin /usr/bin/composer /usr/bin/composer
WORKDIR /app
```

> Desde la carpeta que contiene Dockerfile, ejecutar en PowerShell:

```text
docker build -t desapp-avz/php:8.5 .
docker run --rm --pull=never desapp-avz/php:8.5 php -m
docker run --rm --pull=never desapp-avz/php:8.5 composer --version
```

Validar todas las extensiones PHP de la sección 2; corregir la receta si la imagen base cambia. Guardar la imagen local terminada. Para Node/PHP en contenedores, verificar montaje de archivos Windows, permisos y conectividad; son opciones de recuperación, no sustitutos automáticos de una instalación nativa.

# 4. Recuperación de MySQL con Docker

Si MySQL nativo falla, recuperar primero el servicio. Si no es posible, utilizar el siguiente Compose de contingencia en una carpeta de Operaciones/instructor separada de TaskFlow. Este procedimiento no se entrega como práctica al alumno.

## Archivo compose.yaml

```text
services:
  mysql:
    image: mysql:8.4
    pull_policy: never
    ports:
      - "127.0.0.1:${MYSQL_PORT:-3307}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:?Definir clave local}
      MYSQL_ROOT_HOST: "%"
    volumes:
      - mysql-data:/var/lib/mysql
    restart: "no"
volumes:
  mysql-data:
```

Configurar .env en esa carpeta (no el .env de Laravel), con MYSQL_PORT=3307 y MYSQL_ROOT_PASSWORD con una contraseña local fuerte. No incluir en Git ni copiar credenciales a capturas. El acceso root remoto dentro del contenedor permite TCP desde el host; aislar el proyecto Docker y no publicar el puerto en otras interfaces. No conectar contenedores ajenos a su red.

```text
docker compose up -d
docker compose ps
# Esperar hasta que el servidor acepte conexiones:
mysql -h 127.0.0.1 -P 3307 -u root -p -e "SELECT VERSION();"
```

## Activación y recuperación de datos

- Puerto alternativo 3307 evita colisiones. Si se libera 3306, puede asignarlo explícitamente en el .env de Compose. En caso contrario, ajustar DB_PORT=3307 en .env y .env.testing de TaskFlow; ejecutar php artisan config:clear. Revisar phpunit.xml y verificar ambas bases antes de ejecutar tests.

- Crear las bases y el usuario según el laboratorio 2. La cuenta de aplicación en el contenedor debe admitir la conexión desde el host (el origen no es necesariamente 127.0.0.1); Operaciones verifica CURRENT_USER() y restringe permisos a taskflow y taskflow_test. No cambiar a root como usuario de Laravel.

- Una instancia nueva está vacía. Si existe trabajo anterior, respaldar y restaurar las bases desde una copia autorizada; no ejecutar migraciones destructivas para recuperar datos. El volumen conserva datos entre reinicios; usar docker compose down sin -v. Nunca ejecutar prune --volumes.

> Validar la contingencia antes del curso con un volumen temporal aislado y sin credenciales de alumnos. Detener y eliminar exclusivamente sus recursos temporales; no dejar un segundo servidor ejecutándose al entregar.

# 5. Red y validación de entrega

## Conectividad y puertos

| **Destino / recurso** | **Requisito** |
|---|---|
| 127.0.0.1:3306 | MySQL nativo, escucha local y autenticación real. |
| 127.0.0.1:8000 / 5173 | Libres; Laravel y Vite durante práctica. Mantener un origen consistente. |
| 127.0.0.1:3307 | Libre para contingencia MySQL. |
| HTTPS saliente: 443 | GitHub, Packagist, npm Registry, CDN y descargas requeridas. |
| Docker Hub / CDN | Acceso para descarga y construcción; precargar antes de clase. |

Configurar proxy y CA corporativa para Git, PHP/Composer, npm y Docker cuando aplique. No desactivar verificación TLS. No se requieren puertos entrantes de Internet, nube ni credenciales IA para el alumno. Para demostración del instructor, permitir acceso al proveedor elegido desde su cuenta.

## Validación automática

Entregar junto con esta guía el script validate-environment.ps1. Ejecutar en la sesión del alumno, con Docker iniciado, sin elevar privilegios y sin proyectos en ejecución. Revisar código antes de ejecutar conforme a la política institucional.

```text
powershell.exe -NoProfile -File .\validate-environment.ps1
# Indicar otra cuenta administrativa local si está autorizada:
# ... -MySqlAdmin admin_local
```

El script solicita la contraseña de MySQL por su prompt nativo; no la almacena. Devuelve PASS/FAIL por comprobación y código de salida 0/1. No crea proyectos ni esquemas. Un PASS automático no equivale a aprobación final: comprobar los puntos manuales de abajo.

## Comprobaciones manuales complementarias

- Confirmar servicio MySQL automático tras reinicio, escucha solo local, privilegios y autenticación; confirmar que 3307 está libre y que el volumen de Docker tiene espacio suficiente.

- Abrir VS Code, navegador y Bruno/Postman con el usuario final; confirmar que ejecutan sin licencias o autenticación no previstas. Comprobar proxy, instalación de paquetes y políticas de ejecución.

- Realizar los smoke tests de la sección 6, conservar evidencia y limpiar solo los recursos temporales. Documentar excepciones, versión exacta y responsable.

# 6. Smoke tests y estado de liberación

## Prueba integral de Laravel (solo Operaciones)

En PowerShell, bajo la cuenta del alumno, crear un proyecto temporal único, fuera de labs. No reutilizar una carpeta existente ni instalar TaskFlow. El esqueleto puede inicializar SQLite; este smoke test no sustituye las pruebas MySQL de las prácticas.

```text
$probe = Join-Path $env:TEMP ("desapp-" + [guid]::NewGuid())
composer create-project laravel/laravel $probe "^13.0"
Set-Location $probe
php artisan --version
composer check-platform-reqs
npm.cmd install
npm.cmd run build
php artisan test
php artisan serve --host=127.0.0.1 --port=8000
```

Esperado: Laravel 13.x, dependencias y compilación sin error, pruebas predeterminadas aprobadas. En otra terminal: Invoke-WebRequest http://127.0.0.1:8000 y confirmar status 200. Detener con Ctrl+C, salir de la carpeta y borrar únicamente la ruta temporal registrada. Si composer o npm fallan, detener y corregir antes de seguir.

## Prueba de permisos MySQL y respaldo

En la instancia aislada, un administrador debe crear un esquema y una cuenta temporales con sufijo único, conceder permisos sobre ese esquema, conectarse como esa cuenta por TCP, crear una tabla, insertar/consultar una fila y retirarlos. Registrar resultado; no usar los nombres taskflow ni taskflow_test. Repetir la conexión y escritura en MySQL Docker con 3307 y un volumen temporal: comprobar persistencia tras reinicio, después retirar solo esos recursos. Validar también composer/npm y acceso a archivos desde las imágenes si se pretende usar la contingencia completa.

## Estado inicial exigido

| **Debe existir / funcionar** | **No debe existir** |
|---|---|
| PHP, Composer, Node, npm, Git, VS Code, cliente REST, navegador, MySQL y Docker operativos. | TaskFlow, proyecto final, .env de aplicación, cuentas/claves de IA. |
| $HOME\labs vacío o listo para nuevos proyectos; puertos requeridos y permisos de escritura. | taskflow, taskflow_test y usuario taskflow; bases del proyecto final. |
| Imágenes de respaldo locales y evidencias PASS; credenciales administrativas locales entregadas de forma segura. | Contenedores/volúmenes de prueba activos, claves o contraseñas en repositorios. |

# 7. Checklist de aceptación y referencias

Operaciones marca cada punto después de probar con la cuenta real del alumno. Si falla un requisito, no liberar el ambiente sin corrección o excepción aprobada por instructor y Operaciones.

| **Verificación** | **Estado** |
|---|---|
| Windows 11 x64; CPU, RAM, disco y persistencia aprobados. | [ ] |
| PHP 8.5, Composer 2, Node 24, npm, Git y PATH correctos. | [ ] |
| Extensiones PHP y límites de archivos correctos. | [ ] |
| MySQL 8.4, TCP, permisos y servicio automático tras reinicio. | [ ] |
| Docker Desktop, cuenta, licencia, backend y Compose aprobados. | [ ] |
| Imágenes locales y prueba PHP/Docker; respaldo MySQL validado. | [ ] |
| VS Code, cliente REST, navegador y acceso HTTPS verificados. | [ ] |
| Smoke Laravel: instalación, build, pruebas y HTTP 200. | [ ] |
| Script PASS y checklist manual sin incidencias pendientes. | [ ] |
| Sin proyectos ni bases de laboratorio; credenciales locales seguras. | [ ] |

*Registro de entrega: equipo/VM ____________________ Fecha _____________ Responsable Operaciones ____________________ Instructor ____________________ Versiones/digests y evidencia: ____________________ Incidencias: ______________*

## Fuentes técnicas y descarga oficial

- Laravel 13: https://laravel.com/docs/13.x/deployment

- PHP Windows: https://windows.php.net/download/ \| Composer: https://getcomposer.org/download/

- Herd: https://herd.laravel.com/windows \| Node: https://nodejs.org/en/download

- MySQL: https://dev.mysql.com/downloads/mysql/8.4.html

- MySQL Windows: https://dev.mysql.com/doc/refman/8.4/en/windows-installation.html

- Git: https://git-scm.com/downloads \| VS Code: https://code.visualstudio.com/download

- Bruno: https://www.usebruno.com/downloads \| Chrome: https://www.google.com/chrome/

- Docker Windows: https://docs.docker.com/desktop/setup/install/windows-install/

- Licencia: https://docs.docker.com/subscription-billing/desktop-license/

- VM/VDI: https://docs.docker.com/desktop/setup/vm-vdi/

- Compose: https://docs.docker.com/compose/intro/history/

- Base del diseño: Setup Guide original SUG-[DESAPP_AVZ].docx; guías LAB01 a LAB06 TaskFlow corregidas. Revisar enlaces y versiones al aprovisionar. Las rutas y configuraciones del curso se conservan; la imagen y el script requieren certificación en Windows antes de liberar.

<!-- {% endraw %} -->
