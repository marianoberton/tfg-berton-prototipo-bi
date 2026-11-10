# Prototipo de inteligencia de negocios conversacional y proactiva para una PyME industrial

Trabajo Final de Grado, Licenciatura en Ciencia de Datos, Universidad Siglo 21. Autor: Mariano Berton. Tutor: Víctor Omar Palazzesi. Noviembre de 2026.

## Qué es el prototipo

Es un sistema de inteligencia de negocios para una fábrica de cartón con tres canales de venta (mayorista, tienda en línea y local). Reúne en una sola base los datos del sistema de gestión, del CRM y de la tienda en línea, resuelve la identidad de cada cliente entre esas fuentes, calcula un catálogo gobernado de métricas y, sobre ese catálogo, ofrece un tablero, un agente conversacional que responde en lenguaje natural y un motor de alertas que propone tareas al vendedor responsable. El código fuente permanece en un repositorio privado; este repositorio publica el demo y el tutorial de uso.

## Demo

- Video del recorrido (un minuto, sin sonido): [demo/TP4_demo_Berton_Mariano.mp4](demo/TP4_demo_Berton_Mariano.mp4). Recorre las pantallas de la aplicación instalada en el servidor el 9 de noviembre de 2026, con importes reales y nombres de clientes y de personas reemplazados por seudónimos. No incluye el agente conversacional, cuyas respuestas se reproducen en el Anexo H del informe.
- Aplicación en línea: https://mpbi.161-97-160-180.sslip.io. Requiere una cuenta; como muestra datos reales de la empresa, el usuario para el tribunal se entrega a pedido.

## Tutorial de uso

El recorrido sigue el flujo de trabajo de los usuarios. Entre paréntesis, la historia de usuario del product backlog que cubre cada paso.

1. **Ingreso por rol.** Se entra con usuario y contraseña. Cada rol ve lo que le corresponde: la dueña ve la cartera completa y la administración; la vendedora, su cartera y sus tareas.
2. **Resumen (HU-05).** Ventas netas por canal del último mes cerrado contra el anterior, ventas en riesgo de la cartera mayorista, alertas abiertas por severidad y fecha del último dato de cada fuente.
3. **Cartera mayorista (HU-06 y HU-07).** Clientes ordenados por probabilidad de compra en los próximos 60 días, con su cadencia habitual, su próxima reposición prevista y sus categorías abandonadas.
4. **Ficha de un cliente (HU-08).** Compras, ciclo de reposición por artículo, categorías y alertas abiertas, sin datos personales: la ficha no muestra CUIT, teléfonos ni correos.
5. **Alertas y mis tareas (HU-09).** Cada alerta indica la regla que la disparó, el responsable y la tarea propuesta. Se reconoce o se resuelve desde la pantalla, y queda reflejada en la lista de tareas del responsable.
6. **Chat con el agente (HU-10 y HU-11).** Se le pregunta por una métrica y responde con su valor, su definición y su versión del catálogo; se le pregunta por un cliente y qué ofrecerle; si se le piden datos personales, lo rechaza; si propone una tarea, requiere aprobación y se ejecuta en modo de simulación.
7. **Administración (HU-01 y HU-03).** Estado de cada corrida de carga por fuente y revisión de las sugerencias de identidad: pares de clientes posiblemente duplicados entre el CRM y el sistema de gestión, que una persona aprueba o rechaza uno por uno.

## Instalación y ejecución

Los requisitos son Python 3.12 con el gestor de paquetes uv, Node.js con pnpm y una base PostgreSQL 15 o superior; para la instalación en un servidor se requiere además Docker. Las claves de los servicios externos (HubSpot y el proveedor del modelo de lenguaje) y la cadena de conexión a la base se configuran en un archivo de entorno que no forma parte del repositorio.

Entorno de desarrollo. Después de clonar el repositorio de código y de crear el archivo de entorno, se ejecutan los siguientes comandos desde la carpeta raíz:

```bash
uv sync --all-packages              # dependencias de Python
pnpm install                        # dependencias de la aplicación web
pnpm db:push                        # aplica las migraciones de db/migrations
uv run mpbi seed load               # mapeos de sucursales y de vendedores
uv run mpbi ingest erp              # carga los reportes de data/erp/inbox
uv run mpbi ingest hubspot --since 2025-01-01
uv run mpbi identity                # identidad de clientes
uv run mpbi transform               # vistas materializadas
uv run mpbi model recompra          # evaluación y probabilidad de compra
uv run mpbi alerts run              # evaluación de las reglas de alerta
uv run uvicorn metrics_api.main:app # API de métricas
pnpm --filter web dev               # aplicación web
uv run pytest -q                    # pruebas automáticas
uv run mpbi catalog validate        # validación del catálogo
```

Servidor. La instalación completa en un servidor con Docker se hace con un único guion, que copia el código, construye las imágenes, crea la base y sus roles, aplica las migraciones, levanta los servicios y deja programadas las tareas de carga, alertas y respaldo. Las cuentas de los usuarios se crean con un segundo guion, que recibe el correo y el rol:

```bash
bash deploy/vps/instalar.sh                      # instala o actualiza
deploy/vps/usuario.sh <correo> <rol>             # crea un usuario
bash scripts/pipeline.sh diaria|hubspot|alertas  # corre una tarea a mano
```

La evaluación del agente se ejecuta sobre una base de pruebas con datos sintéticos, nunca sobre la base con datos reales, y requiere la clave del proveedor del modelo de lenguaje.

## Datos

Los datos reales de la empresa son confidenciales y no se publican. Este repositorio no contiene código, datos, claves ni capturas con datos reales.
