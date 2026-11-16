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

## 3. Tabla de amenazas

## 4. Árbol de ataque

## 5. Matriz de riesgo

## 6. Trazabilidad de las 5 amenazas críticas

## 7. Conclusiones
