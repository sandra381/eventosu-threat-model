# EventosU: Modelado de amenazas

Modelado de amenazas del sitio web ficticio **EventosU** en el cual se publica sobre congresos, talleres y conciertos.En donde los usuarios se inscriben, pagan y reciben un código QR de acceso.

**Curso:** Ciberseguridad 
**Catedrático:** Ing. Ing. Lester Monterroso
**Universidad:** Universidad Galileo
**Fecha de entrega:** 11/11/2026

## Integrantes
 
| Nombre | Carné | Correo | Aportes principales |
|--------|-------|--------|---------------------|
| Gabriela Lucia Navarro de Leon| 20000127| gabriela.navarro@galileo.edu |  |
| Sandra Daniela Soria Palma | 20002619 | sandra.soria@galileo.edu|  |
 
> El historial de commits refleja el trabajo de ambos.

## Alcance

Analizamos el diseño de EventosU buscando posibles fallas que afecten el día del evento,por ejemplo, personas sin entradas válidas adentro o personas con entradas que no logran ingresar. Nos enfocamos sobre todo en un atacante oportunista, sin conocimientos avanzados, que quiere colarse sin pagar.

**Dentro del modelo:** Todo lo que el equipo de EventosU diseñaría y operaría: el sitio web (React en CDN), la API (Node.js), la base de datos PostgreSQL, el bucket de archivos, el panel /admin y la app de puerta, junto con los puntos de conexión con Microsoft Entra ID, la pasarela de pagos y el servicio de correo. Se cubren los cinco flujos del caso: registro/login, inscripción, pago, ingreso y gestión.

**Fuera del modelo:** El funcionamiento interno de los servicios de terceros, el procesamiento de tarjetas, la seguridad física de los celulares de puerta y de la red del campus, y las fallas propias del proveedor de nube. Quedan fuera porque no dependen de decisiones de diseño de EventosU.

**Supuestos principales:**
- El webhook de la pasarela llega por HTTPS, pero el diseño no menciona ninguna firma; asumimos que no viene firmado.
- El JWT se guarda en localStorage y el rol del usuario viaja dentro del token.
- El código QR contiene solo el identificador de la inscripción, sin firma.
- El día del evento la conexión en la puerta puede ser inestable.


## Estructura del repositorio
 
```
eventosu-threat-model/
├── README.md              # este archivo
├── model/
│   └── eventosu.json      # modelo de OWASP Threat Dragon
└── docs/
    ├── informe.pdf        # informe final
    └── amenazas.csv       # tabla de amenazas exportada (opcional)
```
## Cómo abrir el modelo

Requisitos: OWASP Threat Dragon, versión de escritorio [versión, 2.6.2], instalada desde sus releases en GitHub. Sistema operativo con el que lo probamos:Windows.




## Resumen del contenido
 

## Recursos utilizados
 
- [OWASP Threat Dragon](https://owasp.org/www-project-threat-dragon/)
- [Threat Modeling Manifesto](https://www.threatmodelingmanifesto.org/)
- [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)