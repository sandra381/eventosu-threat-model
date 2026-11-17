# Modelado de amenazas: EventosU

**Curso:** Ciberseguridad
**Catedrático:** Ing. Lester Monterroso
**Universidad:** Universidad Galileo
**Fecha de entrega:** 17/11/2026

**Integrantes:**
- Gabriela Lucía Navarro de León, 20000127
- Sandra Daniela Soria Palma, 20002619

## 1. Alcance, activos y supuestos

### Alcance

Se analizó el diseño de EventosU buscando fallas que afecten el día del evento, por ejemplo personas sin entrada válida adentro o personas con entrada que no logran ingresar. El enfoque principal es un atacante oportunista, sin conocimientos avanzados, que quiere entrar sin pagar.

**Dentro del modelo:** el sitio web (React en CDN), la API (Node.js), la base de datos PostgreSQL, el bucket de archivos, el panel /admin y la app de puerta, junto con los puntos de conexión con Microsoft Entra ID, la pasarela de pagos y el servicio de correo. Se cubren los cinco flujos del caso: registro/login, inscripción, pago, ingreso y gestión.

**Fuera del modelo:** el funcionamiento interno de los servicios de terceros, el procesamiento de tarjetas, la seguridad física de los celulares de puerta y de la red del campus, y las fallas propias del proveedor de nube, porque no dependen de decisiones de diseño de EventosU.

### Activos

| # | Activo | ¿Por qué es importante? |
|---|--------|-------------------------|
| A1 | Datos de los inscritos | Contienen información personal como nombre, correo, carné y teléfono. Si se filtran, se afecta la privacidad de los usuarios. |
| A2 | Cuentas de usuario y sesiones | Permiten que cada usuario acceda a su cuenta y sus entradas. Si alguien obtiene acceso sin permiso, podría utilizar las entradas de otra persona. |
| A3 | Registro de pagos | Permite verificar quién ha pagado su inscripción. Si alguien modifica estos registros, podría obtener una entrada sin pagar. |
| A4 | Códigos QR | Se utilizan como entradas para los eventos. Si se falsifican o reutilizan, podrían permitir ingresos no autorizados. |
| A5 | Validación de QR en la puerta | Permite comprobar si una entrada es válida o ya fue utilizada. Si falla, se podría impedir el ingreso de personas que sí pagaron o permitir entradas no válidas. |
| A6 | Panel de administración | Permite administrar usuarios, roles y consultar reportes. Si alguien accede sin autorización, podría realizar cambios indebidos. |
| A7 | Afiches y archivos CSV | Los CSV contienen datos personales de los inscritos y los afiches muestran información de los eventos. Si se modifican o exponen, podrían afectar a los usuarios. |
| A8 | Catálogo de eventos | Muestra la información de los eventos disponibles. Si se modifica sin autorización, los usuarios podrían consultar fechas, precios o información incorrecta. |

### Supuestos

Como el caso no define algunos detalles del diseño, se tomaron los siguientes supuestos para el análisis:

- **S1.** El webhook del proveedor de pagos llega por HTTPS, pero no trae una firma, así que la API no tiene forma de comprobar que la notificación realmente viene del proveedor.
- **S2.** El token de sesión (JWT) se guarda en el navegador y dentro del token va el rol del usuario.
- **S3.** El código QR solo tiene el número de inscripción y no lleva una firma, por lo que no se puede comprobar si es auténtico.
- **S4.** El día del evento la conexión a Internet en la puerta puede fallar, y eso afectaría la validación de las entradas.

## 2. DFD nivel 0 y nivel 1

### Nivel 0 (contexto)

En el nivel 0 se representa EventosU como un solo proceso, dentro del límite "Nube de EventosU". A la izquierda se muestran los cinco tipos de usuario que interactúan con el sistema y a la derecha los tres servicios externos que utiliza: Microsoft Entra ID, el proveedor de pagos y el servicio de correo.

![DFD Nivel 0](<img/EventosU - Nivel 0 (contexto).png>)

### Nivel 1

En el nivel 1 se muestran por separado los componentes de EventosU: sitio web, panel de administración (`/admin`), app de puerta, API, base de datos, bucket de archivos y servicios externos.

También se definieron cinco límites de confianza:

- **Navegador del usuario:** contiene el sitio web y el panel `/admin`. Es un entorno que EventosU no controla completamente, por lo que no se debe confiar directamente en la información que envía.
- **Celular de puerta:** contiene la aplicación que utiliza el personal para escanear los códigos QR y consultar su validez.
- **Nube de EventosU:** contiene la API, encargada de procesar las solicitudes y realizar las validaciones del sistema.
- **Red privada de datos:** contiene la base de datos y el bucket de archivos, donde se almacena la información utilizada por EventosU.
- **Servicios de terceros:** incluye Microsoft Entra ID, el proveedor de pagos y el servicio de correo. Son servicios externos que EventosU utiliza, pero no controla directamente.

![DFD Nivel 1](<img/EventosU - Nivel 1.png>)

## 3. Tabla de amenazas STRIDE

| ID | Elemento | STRIDE | Escenario (Quién, cómo, qué obtiene) | Impacto | Probabilidad | Riesgo | Respuesta | Mitigación | Estado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **S-01** | Flujo: Webhook de pago | Spoofing | Un atacante descubre la URL del webhook y manda un mensaje falso diciendo que el pago se hizo, logrando sacar un código QR gratis. | Alto | Alta | Crítico | Mitigar | Ponerle una firma secreta a los mensajes de la pasarela. Si el sistema no ve la firma exacta, rechaza el mensaje. | En diseño |
| **S-02** | Flujo: Cuenta de Organizador | Spoofing | Alguien se hace pasar por un organizador oficial, crea un evento falso y engaña a los estudiantes para que le transfieran el dinero de la inscripción. | Medio | Media | Medio | Mitigar | Pedir validación de identidad antes de darle permisos de organizador a alguien. | En diseño |
| **S-03** | Flujo: Validación de QR | Spoofing | Un atacante muestra en la puerta una foto del QR de un amigo. El guardia lo escanea y el sistema no distingue si es un código real o una foto. | Bajo | Alta | Medio | Aceptar | Se acepta porque el QR es de un solo uso. Para protegernos, el QR lleva un sello digital y el sistema lo marca como usado al instante. Responsable: Coordinación de eventos. | Aceptado |
| **T-01** | Flujo: Validación de QR | Tampering | Un atacante modifica el número de entrada dentro del QR porque este no tiene candado; por lo que podría entrar a un evento VIP pagando uno barato. | Alto | Alta | Crítico | Mitigar | Ponerle un sello digital al QR. Si alguien intenta modificarlo, el sello se rompe y el sistema lo rechaza. | En diseño |
| **T-02** | Flujo: Peticiones Admin | Tampering | Un estudiante abre su gafete digital en el navegador y cambia su rol a admin para entrar a zonas restringidas. | Alto | Media | Alto | Mitigar | El servidor nunca debe creer lo que dice el token; debe consultar la base de datos para revisar el rol real en cada petición. | En diseño |
| **T-03** | Flujo: Peticiones de usuario con JWT | Tampering | El atacante usa el modo desarrollador del navegador para cambiar el precio de un evento de $50 a $1 antes de pagar, obteniendo la entrada barata. | Medio | Media | Medio | Mitigar | El servidor debe ignorar el precio que manda la página web; tiene que buscar el precio real directamente en su base de datos. | En diseño |
| **R-01** | Flujo: Webhook de pago | Repudiation | Un estudiante que no recibió el QR dice que no pagó. Como no hay un registro histórico, la universidad no puede probar lo contrario. | Medio | Media | Medio | Mitigar | Llevar un registro de logs que guarde la fecha, la hora y los datos de cada pago que entra al sistema. | En diseño |
| **R-02** | Proceso: App de Puerta | Repudiation | Un guardia deja pasar a gente sin escanear el QR. Luego lo niega, y como la app no guarda los ingresos manuales, no hay pruebas en su contra. | Bajo | Baja | Bajo | Aceptar | Se confía en el personal contratado. Se controla con supervisión física, cámaras de seguridad y conteo manual. | Aceptado |
| **R-03** | Flujo: Afiche y exportación del CSV | Repudiation | Un organizador descarga el CSV de los inscritos y filtra la información. Al no haber registro de quién descargó el archivo, nadie paga las consecuencias. | Alto | Media | Alto | Mitigar | Anotar en un registro quién descargó el CSV y a qué hora. Además, ponerle una marca de agua invisible con el nombre del organizador. | En diseño |
| **I-01** | Almacén: Bucket de Archivos (CSV) | Info Disclosure | El almacén de archivos quedó público por error. Cualquiera en internet puede adivinar la URL y descargar todos los CSV con datos privados. | Alto | Alta | Crítico | Mitigar | Ponerle candado al almacén. Dar permisos de descarga que duren solo 5 minutos para que nadie más pueda entrar. | En diseño |
| **I-02** | Proceso: Logs de Error | Info Disclosure | Cuando el sistema falla, muestra un error en pantalla que revela contraseñas y rutas internas. El atacante usa eso para atacar la base de datos. | Medio | Media | Medio | Mitigar | Ocultar los errores técnicos en la versión final y solo mostrar un mensaje genérico como "Ups, algo salió mal". | En diseño |
| **I-03** | Flujo: Validación de QR | Info Disclosure | La app de la puerta recibe todos los datos del estudiante al escanear. Si el guardia pierde el celular, la información queda expuesta. | Medio | Baja | Bajo | Mitigar | Hacer que el sistema solo responda si el QR es válido y el nombre. No es necesario enviar teléfonos ni correos. | En diseño |
| **D-01** | Flujo: QR y recuperación de contraseña | Denial of Service | Un atacante usa el botón de "olvidé mi contraseña" mil veces con el correo de alguien. El servicio de correo bloquea la cuenta y la víctima no puede recuperar su contraseña. | Medio | Media | Medio | Mitigar | Poner un límite de peticiones por minuto y usar un Captcha para frenar a los atacantes. | En diseño |
| **D-02** | Flujo: Validación de QR | Denial of Service | En pleno evento, un atacante satura el sistema con escaneos falsos. La red colapsa y la gente hace filas gigantes porque no pueden validar las entradas. | Alto | Media | Alto | Mitigar | Hacer que la app de la puerta descargue la lista de entradas antes del evento, para que pueda funcionar aunque no haya internet. | En diseño |
| **D-03** | Flujo: Peticiones de usuario con JWT | Denial of Service | Un atacante prueba miles de contraseñas robadas en el login, logrando bloquear las cuentas de los estudiantes reales. | Medio | Alta | Alto | Mitigar | Bloquear el acceso de una cuenta por 15 minutos después de 5 intentos fallidos de contraseña. | En diseño |
| **E-01** | Proceso: Panel /admin | Elevation | Un estudiante cambia la URL de la página a /admin, y el sistema lo deja pasar porque el servidor no revisa si de verdad es administrador. | Alto | Media | Alto | Mitigar | Revisar siempre en el servidor el rol del usuario antes de dejarlo entrar al panel de administración. | En diseño |
| **E-02** | Flujo: Peticiones de usuario con JWT | Elevation | El atacante manda una petición para editar su perfil, pero esconde un campo extra "isAdmin: true". Si el sistema no filtra, se vuelve administrador. | Alto | Media | Alto | Mitigar | Crear una lista estricta de los campos que el usuario puede editar. Si manda datos extra, el sistema debe ignorarlos. | En diseño |
| **E-03** | Proceso: JWT sin Firma | Elevation | El sistema solo lee el token, pero no revisa si es original. El atacante se crea un token falso diciendo que es admin y el sistema le cree. | Alto | Baja | Medio | Mitigar | Revisar siempre que la firma del token sea original y válida. Si el sello está roto o es falso, rechazar la conexión. | En diseño |

## 4. Árbol de ataque



## 5. Matriz de riesgo

## 6. Trazabilidad de las 5 amenazas críticas

## 7. Conclusiones
