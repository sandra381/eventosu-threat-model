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

Para entender cómo un atacante podría intentar ingresar a un evento de EventosU sin pagar, se construyó el siguiente árbol de ataque. 

![Arbol de ataque](<img/arbol_de_ataque.drawio.png>)

**Leyenda del diagrama:**
*   **Nodo O (OR):** El atacante solo necesita cumplir con una de las opciones para lograr el objetivo.
*   **Nodo Y (AND):** El atacante debe cumplir con todos los pasos de esa rama para tener éxito.

**Justificación del Camino Más Barato (Rama Marcada en Rojo):**
El camino más barato y rápido para el atacante es la rama derecha: Falsificar el Webhook de pago. Debido a que no requiere una interacción física, un dispositivo en la puerta y una cuenta de estudiante.El atacante solo necesita un computadora , descubrir la URL del webhook y enviar una petición HTTP falsa simulando ser la pasarela de pagos.Como el sistema no verifica ninguna firma de seguridad, el servidor le cree y le genera un código QR válido de inmediato.

## 5. Matriz de riesgo

## 6. Trazabilidad de las 5 amenazas críticas
En esta sección se detallan las cinco amenazas de mayor riesgo identificadas en la matriz y la amenaza aceptada.

### Requisito, tarea y control de verificación

Para las cinco amenazas seleccionadas de EventosU, se utilizaron los requisitos de OWASP ASVS 5.0 como referencia para identificar las medidas de seguridad que debería tener el sistema. Para cada amenaza se explica qué se necesita implementar y cómo se podría comprobar que funcione correctamente.

| ID | Amenaza | Requisito ASVS | Tarea | Cómo se verifica |
|----|---------|----------------|-------|------------------|
| S-01 | Webhook falso de pago | **4.1.5:** Los avisos de pago deben llevar una firma de seguridad que permita comprobar quién los envió, ya que contienen información relacionada con operaciones sensibles. | Es necesario que la API compruebe, revisando su firma, que cada confirmación de pago realmente provenga de la pasarela. En caso de recibir una confirmación falsa, el sistema deberá rechazarla, mantener la inscripción como pendiente de pago y no generar el código QR. | **Prueba manual:** Se utiliza curl para enviar una confirmación de pago falsa, sin firma de seguridad. Después, se debe comprobar que la API rechace la confirmación, que la inscripción continúe pendiente de pago y que no se envíe el código QR al correo del usuario. |
| T-01 | QR alterado | **9.1.1:** La firma de un token debe verificarse antes de utilizar sus datos, para comprobar que la información no haya sido modificada. | El QR debe generarse con una firma que proteja los datos de la inscripción y del evento. Cuando el personal encargado del ingreso escanee un código, la API deberá comprobar que no haya sido modificado, que corresponda al evento, que la inscripción esté pagada y que no se haya utilizado anteriormente. | **Prueba manual:** Se utiliza la aplicación de validación de entradas para escanear un QR con el número de inscripción modificado, otro que pertenezca a un evento diferente, uno de una inscripción sin pagar y otro que ya haya sido utilizado. Se debe comprobar que ninguno permita el ingreso. Además, se probará un QR válido para verificar que el sistema sí autorice la entrada. |
| T-02 | Rol modificado dentro del JWT | **9.1.1 y 9.1.2:** El sistema debe verificar la firma de los tokens y aceptar únicamente aquellos que utilicen algoritmos de seguridad permitidos. | La API debe verificar la firma del JWT antes de utilizar la información del rol. De esta manera, si un estudiante modifica su token para cambiar su rol a administrador, el sistema deberá detectar la modificación y rechazarlo. | **SAST:** Se utiliza Semgrep para identificar posibles partes del código donde se acepten JWT sin verificar su firma. **Prueba manual:** Se modifica el rol de un token de estudiante a administrador y se intenta crear un usuario. Se debe comprobar que la API rechace la solicitud. |
| I-01 | CSV expuesto en un bucket público | **8.2.2:** El sistema debe permitir que únicamente los usuarios autorizados puedan acceder a la información correspondiente. | El bucket debe configurarse como privado para evitar que cualquier persona descargue los archivos CSV. Cuando un organizador necesite la lista de inscritos de su evento, la API deberá comprobar que tenga permiso y generar un enlace de descarga que venza después de unos minutos. | **IaC:** Se utiliza Checkov para verificar que el bucket no permita acceso público y detener el pipeline si se detecta esa configuración. **Prueba manual:** Se intenta descargar un CSV sin autorización, solicitarlo desde la cuenta de otro organizador y utilizar un enlace vencido. En ninguno de estos casos debe permitirse la descarga. |
| E-01 | Acceso a /admin cambiando la URL | **8.2.1:** El sistema debe comprobar que cada usuario tenga los permisos necesarios antes de permitirle utilizar funciones restringidas, como las del panel de administración. | La API debe revisar los permisos del usuario cada vez que intente realizar una acción administrativa. No es suficiente con ocultar la página `/admin`, ya que un estudiante podría intentar acceder directamente desde el navegador. | **Prueba manual:** Se inicia sesión con una cuenta de estudiante y se intenta modificar el precio de un evento desde `/admin`. Se debe comprobar que la API rechace la solicitud por falta de permisos y que el precio del evento no cambie. |

### Amenaza aceptada

**Amenaza aceptada:** S-03 — QR compartido por captura de pantalla

**Respuesta:** Aceptar el riesgo residual.

Se acepta el riesgo de que un usuario comparta o revenda su código QR, ya que en este caso cada entrada permitirá un solo ingreso. Por lo tanto, la primera persona que utilice el QR será la que podrá entrar al evento y, si alguien más intenta utilizarlo después, el sistema le indicará que el código ya fue utilizado.

Se considera que el impacto de este riesgo sería limitado. Además, esto permitiría que el ingreso sea un poco más rápido, porque de lo contrario sería necesario que el personal encargado solicite un documento de identificación a cada asistente, lo que podría generar filas más largas.

Para aceptar este riesgo, también es necesario que el QR esté firmado y que la API compruebe que sea válido y lo marque como utilizado en una sola operación. De esta manera, se evita que dos personas puedan ingresar con el mismo código, incluso si intentan utilizarlo al mismo tiempo. Además, la aplicación de validación de entradas deberá consultar siempre a la API y no podrá aprobar un QR por su cuenta cuando no haya conexión.

Si se presentan varios casos de reventa de entradas, será necesario evaluar nuevamente el riesgo.

**Responsable:** La Coordinación de eventos de la universidad será la encargada de aceptar y supervisar este riesgo, así como darle seguimiento si se presenta algún problema.
## 7. Conclusiones

Con este laboratorio se comprendió que, al realizar un análisis de amenazas, algunos riesgos no se pueden eliminar del todo. En EventosU se decidió aceptar el riesgo de que los usuarios compartan su código QR, con la condición de que cada código permita el ingreso de una sola persona. Para tomar esa decisión se evaluó el impacto en la matriz de riesgo, considerando que algunos riesgos pueden aceptarse cuando su impacto es limitado y las medidas para evitarlos pueden resultar más costosas que sus posibles consecuencias.

Antes de empezar a desarrollar un sistema es necesario analizar sus amenazas, porque los aspectos de seguridad deben definirse desde el diseño para evitar problemas a futuro. En el caso de EventosU, antes de programar habría que establecer que los códigos QR llevarán firma, que la API verificará los permisos de los usuarios antes de permitir acciones restringidas y cómo se validarán las entradas cuando no haya conexión a internet.

Por último, es importante proteger las confirmaciones de pago y los códigos QR. Si la API acepta esa información sin verificarla, un usuario podría obtener una entrada sin pagar o modificar un QR para intentar ingresar a otro evento. Por eso, se propone utilizar firmas de seguridad y comprobar que la información del pago y de la entrada sea válida antes de aceptarla.