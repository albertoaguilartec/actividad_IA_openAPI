# Actividad: De SQL a API REST con IA… y auditar lo que la IA generó

## Propósito y reglas de la actividad

En esta actividad usarás un **agente de IA en la terminal de tu computadora** (OpenCode, Codex, Claude Code u otro similar) para pasar de un esquema de base de datos MySQL a una especificación OpenAPI, a servicios web REST y a un frontend.

**El objetivo no es que el sistema funcione, sino que puedas demostrar que cumple los requerimientos y que sus decisiones de diseño son correctas.** Un código que responde bien en el navegador puede tener errores graves de lógica de negocio, integridad de datos o seguridad. Tu trabajo es encontrarlos, decidir qué hacer con cada uno y justificarlo.

**Forma de trabajo:** individual.

### Uso permitido de IA

- Usar tu agente de IA en terminal para generar la especificación OpenAPI, el código del backend, el frontend y las pruebas.
- Usar un chat de IA (ChatGPT, Gemini, Claude, Copilot, etc.) en otra ventana para **entender conceptos**: por ejemplo, "¿cuándo se usa 409 y cuándo 400?" o "¿qué hace `ON DELETE CASCADE`?".
- Pedirle a la IA que critique o audite su propio código, **después** de haber hecho tu propia auditoría (sección 5).

### Uso no permitido de IA

- Entregar código generado sin haberlo revisado y probado.
- Pedirle a la IA que escriba tu registro de decisiones (`decisiones.md`) o tu reflexión (`reflexion.md`). Esos documentos deben reflejar **tu** criterio.
- Compartir con la IA contraseñas, tokens o datos reales. En esta actividad solo se usan datos de ejemplo.
- Aprobar automáticamente todas las acciones del agente (modos tipo *auto-approve* o *yolo*). Cada cambio y cada comando que el agente quiera ejecutar debe pasar por tu revisión.

### Declaración de uso de IA

En tu entrega incluye, al inicio de `reflexion.md`: qué herramienta y modelo usaste, en qué pasos de la actividad los usaste y qué partes del resultado son tuyas. Adjunta además la bitácora de tus sesiones con el agente (ver sección 3).

***

# 0. Pre-requisitos

La actividad se realiza **en tu computadora (macOS o Windows)**. Todo corre ahí: el agente de IA, MySQL en Docker, el backend y el frontend.

| Herramienta | Para qué se usa | Versión sugerida |
|---|---|---|
| Git | Clonar el repositorio y llevar el historial local de tus cambios | cualquiera reciente |
| Docker Desktop | Correr MySQL en un contenedor | 4.x |
| Python + pip | Backend Flask | 3.10 o superior |
| Node.js + npm | Instalar algunos agentes de IA; generar el backend en Express o el frontend en React + Vite | 22 LTS o superior |
| Agente de IA en terminal | Generar y corregir YAML y código | ver sección 0.2 |

## 0.1 Instala las herramientas

### macOS

1. Instala [Docker Desktop para Mac](https://www.docker.com/products/docker-desktop/) (elige **Apple Silicon** o **Intel** según tu equipo) y ábrelo. El ícono de la ballena debe aparecer activo en la barra de menú.
2. Instala Python, Node y Git (con [Homebrew](https://brew.sh)):
   ```sh
   brew install python node git
   ```
3. Si ya tienes MySQL instalado localmente, el puerto 3306 estará ocupado. Detenlo con `brew services stop mysql`, o usa otro puerto en el `docker run` (`-p 3307:3306`) y exporta `DB_PORT=3307` antes de correr el backend.

### Windows 10/11

1. Instala [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop/) con el backend **WSL 2** (el instalador lo solicita y puede requerir reiniciar). Ábrelo antes de continuar.
2. Instala [Git for Windows](https://git-scm.com/download/win) (incluye **Git Bash**), [Python 3.12](https://www.python.org/downloads/) (marca **"Add python.exe to PATH"** durante la instalación) y [Node.js LTS](https://nodejs.org).
3. Usa **Git Bash** para todos los comandos del tutorial, salvo que se indique otra cosa.
4. En Git Bash, si `docker exec -it ...` muestra *"the input device is not a TTY"*, antepón `winpty`:
   ```sh
   winpty docker exec -it mysql-container mysql -u root -pcontrasena
   ```
   > En PowerShell la redirección `< tienda_simple.sql` no funciona; usa Git Bash.

## 0.2 Instala tu agente de IA en terminal

Instala **uno** de los siguientes. Revisa primero qué cuenta necesitas, porque no todos son gratuitos:

| Agente | Cuenta que necesitas | Instalación en macOS | Instalación en Windows |
|---|---|---|---|
| [OpenCode](https://opencode.ai/docs) | Ninguna propia: se conecta a un proveedor de modelos. Puedes usar tu suscripción de **GitHub Copilot** (gratuita para estudiantes verificados en [GitHub Education](https://education.github.com)) o una API key de otro proveedor. | `curl -fsSL https://opencode.ai/install \| bash` | `npm install -g opencode-ai` |
| [Codex CLI](https://github.com/openai/codex) | ChatGPT Plus o superior (Business, Edu), o una API key de OpenAI. | `curl -fsSL https://chatgpt.com/codex/install.sh \| sh` | `npm install -g @openai/codex` |
| [Claude Code](https://code.claude.com/docs/en/setup) | Claude Pro, Max, Team o Enterprise, o una API key de Anthropic Console. El plan gratuito no lo incluye. | `curl -fsSL https://claude.ai/install.sh \| bash` | En PowerShell: `irm https://claude.ai/install.ps1 \| iex` |
| [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/set-up/install-copilot-cli) | Cualquier plan de GitHub Copilot, incluido Copilot Student. | `npm install -g @github/copilot` | `npm install -g @github/copilot` |

Después de instalarlo, ábrelo una vez (`opencode`, `codex`, `claude` o `copilot`) e inicia sesión siguiendo las instrucciones en pantalla. En OpenCode, la conexión con GitHub Copilot se hace con el comando `/connect`.

## 0.3 Clona el repositorio y prepara el proyecto

1. Clona el repositorio de la actividad y entra a la carpeta:
   ```sh
   git clone https://github.com/albertoaguilartec/actividad_IA_openAPI.git
   cd actividad_IA_openAPI
   ```
   No necesitas subir nada a GitHub: tu trabajo se queda en tu computadora y se entrega en Canvas (sección 8).
2. Configura tu nombre y correo en Git, para poder hacer commits locales (solo la primera vez):
   ```sh
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu_matricula@tec.mx"
   ```
3. Crea y activa un entorno virtual de Python:
   ```sh
   # macOS
   python3 -m venv .venv
   source .venv/bin/activate

   # Windows (Git Bash)
   python -m venv .venv
   source .venv/Scripts/activate
   ```
4. Instala las dependencias de Python:
   ```sh
   pip install -r requirements.txt
   ```
5. Crea la carpeta donde guardarás todo tu trabajo:
   ```sh
   mkdir mi_solucion
   ```

Cada vez que abras una terminal nueva para el backend, entra a la carpeta del proyecto y vuelve a activar el entorno virtual (paso 3).

A partir de aquí, **abre tu agente de IA dentro de la carpeta `actividad_IA_openAPI`**. Así puede leer los archivos del proyecto (`tienda_simple.sql`, `AIGenerated/`, etc.).

### ¿Por qué hacer commits si no vas a subir nada?

Git funciona perfectamente sin GitHub. Los commits locales te sirven para revisar con `git diff` lo que cambió el agente, regresar a una versión anterior si algo sale mal y, al final, entregar tu historial de trabajo como evidencia (sección 8).

## 0.4 Verificación

```sh
git --version
opencode --version   # o: codex --version / claude --version / copilot --version
docker --version && docker run --rm hello-world
python3 --version    # en Windows: python --version
pip show flask flask-cors mysql-connector-python
node --version
```

Si tu agente responde con su versión, `hello-world` imprime *"Hello from Docker!"* y `pip show` lista los tres paquetes, estás listo para la sección 1.

***

# 1. Base de datos ejemplo

## Configuración de MySQL

> Ejecuta los comandos de esta sección en la terminal de macOS o en Git Bash (Windows), dentro de la carpeta `actividad_IA_openAPI`.

### Iniciar la instancia de MySQL en Docker

```sh
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=contrasena -e MYSQL_DATABASE=tienda_simple -p 3306:3306 -d mysql:8.4
```

### Conectarse al contenedor a través de la herramienta de línea de comandos para generar las tablas

Espera unos segundos a que MySQL termine de arrancar y ejecuta:

```sh
docker exec -i mysql-container mysql -u root -pcontrasena tienda_simple < tienda_simple.sql
```

Esto deberá crear las tablas necesarias e insertar algunos datos de ejemplo.

### Verificar que las tablas estén creadas y tengan información

```sh
docker exec -it mysql-container mysql -u root -pcontrasena
```

#### Dentro de mysql ejecuta

```sql
USE tienda_simple;
SELECT * FROM clientes;
SELECT * FROM productos;
SELECT * FROM pedidos;
```

Para salir escribe *quit* y presiona Enter.

### Restaurar la base de datos

Las pruebas de las secciones 5 a 7 crean y borran registros. Cuando quieras regresar la base de datos a su estado original:

```sh
docker exec -i mysql-container mysql -u root -pcontrasena -e "DROP DATABASE tienda_simple; CREATE DATABASE tienda_simple;"
docker exec -i mysql-container mysql -u root -pcontrasena tienda_simple < tienda_simple.sql
```

### Dependencias de Python

Las dependencias del backend (`flask`, `flask-cors` y `mysql-connector-python`) ya se instalaron en el **paso 0**. Si te faltó, ejecuta:

```sh
pip install -r requirements.txt
```

***

# 2. Generar la especificación OpenAPI a partir del archivo SQL usando IA

## ¿Qué es OpenAPI 3.0?

**OpenAPI 3.0** es un estándar abierto para describir APIs REST mediante un archivo de texto en formato YAML o JSON. Funciona como un contrato entre el backend y cualquier cliente que lo consuma: define qué endpoints existen, qué parámetros aceptan, qué respuestas regresan y qué esquemas de datos manejan.

Un archivo OpenAPI típico tiene esta estructura:

```yaml
openapi: 3.0.0
info:
  title: Mi API
  version: 1.0.0
servers:
  - url: http://localhost:3000/api/v1
paths:
  /clientes:
    get:
      summary: Lista todos los clientes
      responses:
        '200':
          description: OK
components:
  schemas:
    Cliente:
      type: object
      properties:
        id_cliente:
          type: integer
        nombre:
          type: string
```

Las tres secciones clave son:

- **`paths`**: declara cada endpoint con su método HTTP, parámetros y respuestas posibles.
- **`components/schemas`**: define los modelos de datos reutilizables (equivalente a tus tablas MySQL).
- **`servers`**: indica la URL base donde vive la API.

La ventaja principal es que este archivo es legible por humanos y por herramientas: generadores de código, plataformas de documentación como Swagger UI y herramientas de prueba como Postman o Schemathesis pueden consumirlo directamente. En esta actividad lo usaremos como puente entre el esquema de base de datos y el código de los servicios web.

## Cómo trabajar con tu agente de IA

Abre tu agente en la carpeta del proyecto. A diferencia de un chat, el agente **puede leer los archivos del repositorio, crear y modificar archivos y ejecutar comandos**. Por eso los prompts de esta guía le indican qué archivo leer y dónde guardar el resultado, en lugar de pedirte que copies y pegues.

Reglas de trabajo con el agente:

- **Revisa cada cambio antes de aceptarlo.** Lee el diff que te propone el agente.
- **Revisa cada comando antes de permitir que se ejecute.** Pregúntate qué hace y si es necesario. Ten especial cuidado con comandos que borren archivos o datos.
- **Verifica que solo haya modificado lo que le pediste.** Usa `git status` y `git diff`.
- **Guarda tu bitácora.** Al terminar cada sesión, exporta la conversación (en Claude Code y OpenCode con el comando `/export`; en otras herramientas, copia la conversación) y guárdala en `mi_solucion/bitacora_ia.md`. Puedes agregar varias sesiones al mismo archivo.

## Generar la especificación para solo UNA tabla (para empezar)

Si quieres empezar con algo pequeño, genera primero la especificación CRUD de una sola tabla:

**Prompt para tu agente:**
```
Lee la definición de la tabla clientes en tienda_simple.sql y genera la especificación
OpenAPI 3.0 en formato YAML para una API REST con operaciones CRUD completas sobre esa tabla.
Guárdala en mi_solucion/openapi_clientes.yaml.

Requisitos:
* Incluye los endpoints: GET /clientes, GET /clientes/{id}, POST /clientes, PUT /clientes/{id}, DELETE /clientes/{id}
* Define los schemas Cliente y ClienteInput (sin id_cliente ni created_at para creación/edición)
* Incluye respuestas para los códigos HTTP: 200, 201, 204, 400, 404 y 500
* Agrega ejemplos de request body y response para cada operación
* El servidor base debe ser http://localhost:3000/api/v1
```

## Generar la especificación para TODAS las tablas

**Prompt para tu agente:**

```
Lee el archivo tienda_simple.sql y genera la especificación OpenAPI 3.0 en formato YAML
para una API REST con operaciones CRUD completas para cada tabla. Guárdala en
mi_solucion/openapi.yaml.

Requisitos:
* Por cada tabla genera los endpoints: GET /{recurso}, GET /{recurso}/{id}, POST /{recurso}, PUT /{recurso}/{id}, DELETE /{recurso}/{id}
* Los recursos deben llamarse clientes, productos y pedidos
* Define un schema por cada tabla (ej. Cliente, Producto, Pedido) y su variante Input sin campos autogenerados como id y created_at
* Respeta las relaciones entre tablas: pedidos referencia a clientes y productos mediante sus IDs
* Incluye respuestas para los códigos HTTP: 200, 201, 204, 400, 404 y 500
* Define un schema reutilizable Error con campos code y message, y úsalo en todas las respuestas de error
* Agrega ejemplos de request body y response para cada operación de cada recurso
* El servidor base debe ser http://localhost:3000/api/v1
* Agrupa los endpoints por tag: uno por tabla (Clientes, Productos, Pedidos)
```

Algunos ajustes opcionales que puedes agregar al prompt:

* _"Usa autenticación Bearer Token (JWT)"_, si necesitas seguridad.
* _"Agrega paginación con parámetros `page` y `limit` en los `GET` de listas"_, para listas grandes.

> **Para explorar:** pide la misma especificación dos veces, en sesiones distintas, y compara los resultados con `git diff`. ¿Qué tan estable es la salida de la IA? ¿Qué decisiones cambió de una vez a otra?

## Validar la sintaxis del YAML

Usa las siguientes herramientas para validar tu archivo:

- https://oas-validation.com
- https://swapcode.ai/swagger-validator

## Validar el contenido: tabla de trazabilidad

Los validadores anteriores solo revisan que el YAML esté **bien escrito**, no que describa **correctamente** tu base de datos. Para eso, llena una tabla de trazabilidad en `mi_solucion/trazabilidad.md`, con una fila por cada columna de las tres tablas:

| Columna SQL | Restricción en SQL | ¿Cómo aparece en el YAML? | ¿Se valida en el código? (sección 5) | Observación |
|---|---|---|---|---|
| clientes.email | `VARCHAR(150) NOT NULL UNIQUE` | | | |
| productos.precio | `DECIMAL(10,2) NOT NULL` | | | |
| pedidos.id_cliente | `FK → clientes, ON DELETE CASCADE` | | | |
| … | | | | |

Preguntas que te ayudan a llenarla: ¿el YAML marca como obligatorios los campos `NOT NULL`? ¿Refleja los límites de longitud? ¿Cómo expresa la restricción `UNIQUE`? ¿Qué pasa con las reglas `ON DELETE` de las llaves foráneas, se pueden expresar en OpenAPI?

***

# 3. Generar los servicios web a partir del archivo YAML usando IA

## Ventajas y desventajas

Es una práctica muy común y generalmente recomendable.

**Ventajas**

La especificación OpenAPI es una fuente de verdad estructurada y sin ambigüedades, lo que hace que la IA genere código bastante confiable a partir de ella. En la práctica obtenemos:

* **Consistencia**: los nombres de campos, tipos de datos y rutas coinciden con el YAML, sin errores de transcripción.
* **Velocidad**: podemos tener un servidor funcional con validaciones, modelos y rutas en minutos. El código repetitivo (manejo de errores, estructura de carpetas) sale casi gratis.
* **Multi-lenguaje**: el mismo YAML puede producir código en Node/Express, FastAPI, Spring Boot, Laravel, etc., sin reescribir el contrato.
* **Generación de clientes y pruebas**: además del servidor, podemos generar SDKs para el frontend, colecciones de Postman o pruebas automáticas desde el mismo archivo.

**Desventajas y riesgos reales**

El código generado rara vez está listo para producción sin revisión. Los problemas más frecuentes son:

* **Lógica de negocio ausente**: la IA genera la estructura, pero las reglas del negocio ("no se puede eliminar un cliente con pedidos activos") no viven en el YAML y hay que agregarlas a mano.
* **Seguridad superficial**: autenticación, autorización por roles y sanitización de entradas suelen generarse como placeholders o se omiten si el YAML no los especifica con detalle.
* **Código no idiomático**: a veces el resultado funciona pero no sigue las convenciones del framework, lo que genera deuda técnica.
* **Falsa confianza**: el mayor riesgo es asumir que, porque compila y responde bien en Postman, está listo. Los casos límite (concurrencia, transacciones, errores del driver de MySQL) necesitan atención humana.

**Recomendación práctica**

Usarlo, pero con una estrategia en capas:

1. **Genera con IA** el código base completo (rutas, controladores, modelos, middleware de validación).
2. **Revisa y ajusta** la conexión a base de datos, el manejo de errores y las transacciones.
3. **Agrega a mano** la lógica de negocio y la seguridad.
4. **Genera también** las pruebas desde el mismo YAML y úsalas para validar el código generado.

El YAML es suficientemente completo para que una IA produzca alrededor del 70% del trabajo real. Ese porcentaje es el que más tiempo lleva hacer a mano, así que la ganancia al usar IA es muy buena. **El 30% restante es justo lo que vas a buscar en las secciones 5 a 7.**

## Prompts para tu agente

**Python**

```
Lee mi_solucion/openapi.yaml y genera una API REST funcional con Flask y MySQL.
Guárdala en mi_solucion/main.py.

Requisitos:
- Flask + mysql-connector-python (ya están en requirements.txt)
- Un solo archivo main.py
- Implementa todos los endpoints del YAML con sus rutas,
  métodos HTTP y códigos de respuesta exactos
- Usa variables de entorno para la conexión a MySQL
  (DB_HOST, DB_USER, DB_PASSWORD, DB_NAME, DB_PORT)
- Si un recurso no existe, responde con 404
- Habilita CORS para todos los orígenes usando flask-cors
- El servidor debe escuchar en el puerto 3000
```

---

**Node.js**

```
Lee mi_solucion/openapi.yaml y genera una API REST funcional con Express y MySQL.
Guárdala en mi_solucion/node/index.js, con su package.json.

Requisitos:
- Express + mysql2
- Un solo archivo index.js
- Implementa todos los endpoints del YAML con sus rutas,
  métodos HTTP y códigos de respuesta exactos
- Usa variables de entorno para la conexión a MySQL
  (DB_HOST, DB_USER, DB_PASSWORD, DB_NAME, DB_PORT)
- Si un recurso no existe, responde con 404
- Habilita CORS para todos los orígenes usando el paquete cors
- El servidor debe escuchar en el puerto 3000
```

## Ejecutar tu backend

Con el contenedor de MySQL corriendo:

```sh
python mi_solucion/main.py
# o, si elegiste Node.js:
cd mi_solucion/node && npm install && node index.js
```

El servidor escucha en el puerto **3000**. Pruébalo desde otra terminal:

```sh
curl http://localhost:3000/api/v1/clientes
```

La conexión a MySQL usa estas variables de entorno: `DB_HOST=localhost`, `DB_USER=root`, `DB_PASSWORD=contrasena`, `DB_NAME=tienda_simple`, `DB_PORT=3306`. Si cambiaste el puerto en el `docker run`, expórtalo antes de arrancar, por ejemplo `export DB_PORT=3307`.

**Este es el código que auditarás en la sección 5.** Haz un commit local para conservarlo tal como lo generó la IA:

```sh
git add mi_solucion && git commit -m "Código generado por la IA"
```

## Backend de referencia

En la carpeta `AIGenerated/` hay otro backend Flask generado con IA (`ws.py`) a partir de `AIGenerated/openapi.yaml`, con el mismo esquema de base de datos. Lo usarás en la sección 5 para comparar: dos IAs con el mismo punto de partida no cometen los mismos errores. Se ejecuta igual:

```sh
python AIGenerated/ws.py
```

Solo puede haber un servidor a la vez en el puerto 3000; detén uno con `Ctrl+C` antes de iniciar el otro.

***

# 4. Generar el frontend a partir de los servicios web usando IA

Una vez que tienes los servicios web funcionando, el mismo YAML de OpenAPI sirve como base para que la IA genere un cliente web. Los prompts siguientes asumen que el servidor corre en `http://localhost:3000/api/v1`.

> El ejemplo `AIGenerated/index.html` ya apunta a `http://localhost:3000/api/v1` en la constante `API_BASE_URL`.

## HTML + JavaScript (sin frameworks)

```
Lee mi_solucion/openapi.yaml y genera una aplicación web funcional usando solo
HTML, CSS y JavaScript vanilla. Guárdala en mi_solucion/index.html.

Requisitos:
- Un solo archivo index.html (CSS y JS embebidos en el mismo archivo)
- Una sección por cada recurso (Clientes, Productos, Pedidos) navegable mediante tabs o un menú lateral
- Por cada recurso implementa:
  - Tabla para listar todos los registros (GET /{recurso})
  - Formulario para crear un nuevo registro (POST /{recurso})
  - Formulario para editar un registro existente (PUT /{recurso}/{id})
  - Botón para eliminar un registro con confirmación (DELETE /{recurso}/{id})
- Usa fetch() para todas las llamadas HTTP con la base URL definida en una constante
  API_BASE_URL = 'http://localhost:3000/api/v1'
- Muestra mensajes de éxito y error en pantalla (no solo console.log)
- No uses librerías externas; solo HTML5, CSS y JS nativos del navegador
```

---

## React + Vite

```
Lee mi_solucion/openapi.yaml y genera una aplicación React funcional usando Vite
como bundler, en la carpeta mi_solucion/frontend.

Requisitos:
- Estructura de archivos:
  - src/api/        → un módulo por recurso (clientesApi.js, productosApi.js, pedidosApi.js)
                      con funciones async que encapsulan los fetch() a /api/v1
  - src/components/ → componentes reutilizables: tabla genérica, formulario, modal de confirmación
  - src/pages/      → una página por recurso (ClientesPage, ProductosPage, PedidosPage)
  - App.jsx         → enrutamiento con React Router entre las tres páginas
- Por cada recurso implementa las operaciones CRUD completas:
  GET (listar), GET por id (detalle), POST (crear), PUT (editar), DELETE (eliminar con confirmación)
- Manejo de estado con useState y useEffect; sin librerías de estado global
- Muestra estados de carga (loading) y mensajes de error en la UI
- Usa solo Tailwind CSS para estilos (configurado en el proyecto Vite)
- El archivo vite.config.js debe incluir un proxy de /api hacia http://localhost:3000
  para evitar problemas de CORS en desarrollo
```

> **Nota sobre el proxy en Vite:** configurar el proxy en `vite.config.js` evita los errores de CORS durante el desarrollo sin tener que modificar el backend. En producción se resuelve a nivel de servidor web (nginx, caddy, etc.).

***

# 5. Auditoría del código generado

Esta es la parte central de la actividad. Vas a auditar **el backend que generaste con tu agente** (`mi_solucion/main.py` o `mi_solucion/node/index.js`) y **tu especificación** (`mi_solucion/openapi.yaml`) contra el esquema de la base de datos `tienda_simple.sql`.

Tu código puede tener muchas fallas o pocas; eso depende del modelo y de tu prompt. Lo que se evalúa es que revises a fondo las cinco categorías y respaldes cada conclusión con evidencia. "Revisé este caso, se comporta correctamente y lo comprobé así" también es un hallazgo válido.

## 5.1 Tu auditoría (sin IA)

Lee tu código y pruébalo con `curl`, Postman o el frontend. Usa estas preguntas guía:

**Lógica de negocio**
- ¿Quién debería calcular el `total` de un pedido: el cliente que llama a la API o el servidor?
- ¿Qué debería pasar con el inventario (`stock`) cuando alguien hace un pedido?
- Si dos operaciones de base de datos deben ocurrir juntas, ¿qué pasa si la segunda falla?

**Integridad de datos**
- ¿Qué pasa con los pedidos de un cliente cuando lo borras? ¿Y si intentas borrar un producto que tiene pedidos? ¿Las dos políticas tienen sentido para una tienda? ¿La API le avisa a quien la usa?
- ¿Qué pasa si actualizas un registro con `PUT` y no envías todos sus campos?

**Validación de entradas**
- ¿Qué valores absurdos acepta la API? Prueba cantidades, precios y existencias negativas o en cero, textos muy largos y tipos de dato equivocados.

**Semántica HTTP**
- ¿Cada situación de error devuelve el código HTTP adecuado? Por ejemplo, ¿qué código debería devolver un email duplicado?
- ¿Las respuestas coinciden con lo que promete el YAML?

**Seguridad**
- ¿Qué información ve quien llama a la API cuando ocurre un error?
- ¿Cómo está configurado el servidor al arrancar? ¿Sería aceptable en producción?
- ¿Qué orígenes pueden llamar a la API desde un navegador?

Registra cada hallazgo en `mi_solucion/decisiones.md`:

| # | Hallazgo | Evidencia (cómo lo comprobé) | Decisión | Justificación |
|---|---|---|---|---|
| 1 | | comando `curl` y respuesta obtenida | Aceptar / Modificar / Rechazar | |
| … | | | | |

- **Aceptar:** el comportamiento es correcto o es un riesgo que decides asumir; explica por qué.
- **Modificar:** hay que corregirlo; lo harás en la sección 7.
- **Rechazar:** el enfoque de la IA no sirve y debe replantearse desde el diseño; explica la alternativa.

**Antes de continuar, haz un commit** de `decisiones.md` (`git add mi_solucion/decisiones.md && git commit -m "Auditoría propia"`). El commit queda como evidencia de que tu auditoría fue tuya.

## 5.2 Auditoría de la IA y comparación

Ahora pídele a tu agente (ajusta el nombre del archivo si usaste Node.js):

```
Audita mi_solucion/main.py contra tienda_simple.sql y mi_solucion/openapi.yaml.
Busca errores de lógica de negocio, integridad de datos, validación de entradas,
códigos HTTP y seguridad. Para cada hallazgo indica la línea, el riesgo y cómo
comprobarlo. No modifiques ningún archivo.
```

Agrega al final de `decisiones.md` una sección **"Comparación con la auditoría de la IA"**:

- ¿Qué encontró la IA que tú no?
- ¿Qué encontraste tú que la IA no?
- ¿Alguno de sus hallazgos es incorrecto o exagerado? ¿Cómo lo comprobaste?
- La misma IA que generó el código ahora lo audita. ¿Qué tan confiable es que una herramienta revise su propio trabajo?

Si incorporas hallazgos nuevos a tu tabla, márcalos como "(detectado por la IA)".

## 5.3 Comparación con el backend de referencia

Aplica las mismas preguntas guía al backend de referencia `AIGenerated/ws.py`; no necesitas una auditoría completa, basta con revisar cada pregunta. Agrega a `decisiones.md` una sección **"Comparación con AIGenerated/ws.py"**:

- ¿Qué fallas tiene `ws.py` que tu código no tiene, y al revés?
- ¿A qué atribuyes las diferencias: al modelo, al prompt, a la especificación OpenAPI de la que partió cada uno?
- ¿Alguna falla de `ws.py` te hizo volver a revisar tu propio código y encontrar algo que se te había pasado?

***

# 6. Pruebas automáticas contra el contrato

[Schemathesis](https://schemathesis.readthedocs.io) lee una especificación OpenAPI y genera cientos de peticiones automáticamente para buscar errores del servidor, respuestas que no cumplen el esquema y códigos HTTP no documentados.

Con **tu** backend corriendo, prueba tu propia especificación:

```sh
pip install schemathesis==4.30.1
schemathesis run mi_solucion/openapi.yaml --url http://localhost:3000/api/v1 | tee mi_solucion/schemathesis_antes.txt
```

> **Para explorar:** corre también `schemathesis run AIGenerated/openapi.yaml --url http://localhost:3000/api/v1` con el backend de referencia (`AIGenerated/ws.py`) y compara los resúmenes.

Analiza el resumen:

- ¿Qué tipos de falla encontró? ¿Coinciden con tu auditoría?
- ¿Alguna falla es un problema del **código** y otra un problema del **contrato** (el YAML)? ¿Cómo lo distingues?
- ¿Qué tipo de errores **no** puede encontrar una herramienta como esta? (Piensa en las preguntas de lógica de negocio de la sección 5.)

> Schemathesis crea y borra registros reales. Al terminar, restaura la base de datos (sección 1).

***

# 7. Corrección y reflexión

## 7.1 Corrección

1. Antes de corregir, guarda una copia del código tal como lo generó la IA:
   ```sh
   cp mi_solucion/main.py mi_solucion/main_original.py
   # si usaste Node.js:
   cp mi_solucion/node/index.js mi_solucion/node/index_original.js
   ```
2. Corrige tu backend (`mi_solucion/main.py` o `mi_solucion/node/index.js`) y, si hace falta, tu `mi_solucion/openapi.yaml`, en los hallazgos que marcaste como **Modificar** o **Rechazar**. Puedes hacerlo a mano o con tu agente. Si usas el agente, revisa cada cambio con `git diff` antes de aceptarlo.
3. Haz un commit local por cada corrección (o grupo de correcciones relacionadas), con un mensaje que mencione el número de hallazgo. Por ejemplo: `git add mi_solucion && git commit -m "Hallazgo 3: validar cantidad mayor a cero"`.
4. Restaura la base de datos, ejecuta tu versión corregida y vuelve a correr Schemathesis:
   ```sh
   python mi_solucion/main.py   # o tu backend de Node.js
   # en otra terminal:
   schemathesis run mi_solucion/openapi.yaml --url http://localhost:3000/api/v1 | tee mi_solucion/schemathesis_despues.txt
   ```
5. Si cambiaste el YAML, explica en `decisiones.md` por qué: corregir el contrato para que coincida con el código puede ser correcto o puede ser esconder un problema.

## 7.2 Reflexión

Escribe `mi_solucion/reflexion.md` (máximo una página). Empieza con tu **declaración de uso de IA** (ver "Propósito y reglas") y después responde:

1. ¿En qué partes de la actividad la IA fue confiable y en cuáles no? Da ejemplos concretos de tu `decisiones.md`.
2. Si solo hubieras verificado que el sistema "funciona", ¿qué habrías entregado? ¿Qué consecuencias tendría eso en una tienda real?
3. ¿Qué le pedirías diferente a la IA la próxima vez para evitar los problemas que encontraste? ¿Basta con un mejor prompt o hay cosas que siempre tendrás que revisar tú?

***

# 8. Entregables y evaluación

## Entregables

Todo va dentro de la carpeta `mi_solucion/` y se entrega en **Canvas** como un solo archivo `.zip`. No subas nada a GitHub.

| Archivo | Descripción |
|---|---|
| `openapi.yaml` | Especificación generada con tu agente (sección 2), con las correcciones de la sección 7 si las hubo |
| `trazabilidad.md` | Tabla de trazabilidad SQL → YAML → código (secciones 2 y 5) |
| `main_original.py` y `main.py` (o `node/index_original.js` y `node/index.js`) | Backend tal como lo generó la IA (sección 3) y ya corregido (sección 7) |
| `index.html` o `frontend/` | Frontend generado con tu agente (sección 4) |
| `decisiones.md` | Hallazgos de la auditoría con evidencia, decisión y justificación, más la comparación con la auditoría de la IA y con el backend de referencia (sección 5) |
| `schemathesis_antes.txt` y `schemathesis_despues.txt` | Resultados de las pruebas automáticas antes y después de corregir (secciones 6 y 7) |
| `reflexion.md` | Declaración de uso de IA y reflexión (sección 7) |
| `bitacora_ia.md` | Sesiones exportadas de tu agente de IA |
| `historial_git.txt` | Historial de tus commits locales (ver abajo) |

### Cómo preparar la entrega

1. Exporta tu historial de commits. Muestra el orden y la hora de tu trabajo, por ejemplo que hiciste tu auditoría antes de pedirle la suya a la IA:
   ```sh
   git log --stat --date=iso > mi_solucion/historial_git.txt
   ```
2. Si generaste código con Node.js, **borra las carpetas `node_modules`** dentro de `mi_solucion/` (pesan cientos de MB y se recrean con `npm install`).
3. Comprime la carpeta `mi_solucion` con el nombre `<tu-matrícula>_actividad_IA_openAPI.zip`:
   - **macOS:** en Finder, clic derecho sobre `mi_solucion` → **Comprimir**, y renombra el archivo; o en la terminal: `zip -r <tu-matrícula>_actividad_IA_openAPI.zip mi_solucion`
   - **Windows:** en el Explorador de archivos, clic derecho sobre `mi_solucion` → **Enviar a → Carpeta comprimida (en zip)**, y renombra el archivo.
4. Sube el `.zip` a la tarea correspondiente en Canvas.

## Criterios de evaluación

| Criterio | Descripción | Ponderación |
|---|---|---|
| Trazabilidad de requerimientos | La tabla relaciona cada restricción del esquema SQL con el YAML y el código, e identifica correctamente qué se perdió en cada paso. | 15% |
| Calidad de la auditoría | La auditoría cubre las cinco categorías, cada conclusión está respaldada por evidencia reproducible (comandos y respuestas) y se compara de forma crítica con la auditoría de la IA y con el backend de referencia. Se evalúa la profundidad de la revisión, no el número de fallas encontradas. | 25% |
| Justificación de decisiones | Cada decisión (aceptar, modificar o rechazar) se sustenta en argumentos de la disciplina: integridad de datos, semántica HTTP, seguridad y reglas del negocio. | 20% |
| Funcionamiento después de la corrección | El backend corregido resuelve los hallazgos marcados, no rompe lo que ya funcionaba y lo demuestra con las pruebas de Schemathesis y `curl`. | 15% |
| Uso intencional y fundamentado de la IA | Los prompts son claros, el agente se usa con propósito, sus resultados se verifican y se compara de forma crítica su auditoría con la propia. | 15% |
| Uso responsable de la IA | Declara con transparencia el uso de IA, entrega la bitácora, distingue sus aportaciones de las de la IA y respeta los usos permitidos. | 10% |
| | **Total** | **100%** |
