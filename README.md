# EventosU: Modelado de amenazas

Modelado de amenazas del sitio web ficticio EventosU, en el que la universidad publica congresos, talleres y conciertos. Los usuarios se inscriben, pagan y reciben un código QR de acceso.

**Curso:** Ciberseguridad
**Catedrático:** Ing. Lester Monterroso
**Universidad:** Universidad Galileo
**Fecha de entrega:** 17/11/2026

## Integrantes

| Nombre | Carné | Correo | 
|--------|-------|--------|
| Gabriela Lucía Navarro de León | 20000127 | gabriela.navarro@galileo.edu | 
| Sandra Daniela Soria Palma | 20002619 | sandra.soria@galileo.edu |

> El historial de commits refleja el trabajo de ambas.

## Alcance

Analizamos el diseño de EventosU buscando posibles fallas que afecten el día del evento, por ejemplo, personas sin entradas válidas adentro o personas con entradas que no logran ingresar. Nos enfocamos sobre todo en un atacante oportunista, sin conocimientos avanzados, que quiere colarse sin pagar.

**Dentro del modelo:** todo lo que el equipo de EventosU diseñaría y operaría: el sitio web (React en CDN), la API (Node.js), la base de datos PostgreSQL, el bucket de archivos, el panel /admin y la app de puerta, junto con los puntos de conexión con Microsoft Entra ID, la pasarela de pagos y el servicio de correo. Se cubren los cinco flujos del caso: registro/login, inscripción, pago, ingreso y gestión.

**Fuera del modelo:** el funcionamiento interno de los servicios de terceros, el procesamiento de tarjetas, la seguridad física de los celulares de puerta y de la red del campus, y las fallas propias del proveedor de nube. Quedan fuera porque no dependen de decisiones de diseño de EventosU.

**Supuestos principales:**
- El webhook de la pasarela llega por HTTPS, pero el diseño no menciona ninguna firma; asumimos que no viene firmado.
- El JWT se guarda en localStorage y el rol del usuario viaja dentro del token.
- El código QR contiene solo el identificador de la inscripción, sin firma.
- El día del evento la conexión en la puerta puede ser inestable.
- La app de puerta permite registrar un ingreso manual cuando no se puede escanear el QR.

## Estructura del repositorio

```
eventosu-threat-model/
├── README.md              # este archivo
├── model/
│   └── eventosu.json      # modelo de OWASP Threat Dragon
└── docs/
    ├── informe.md         # informe en markdown
    ├── informe.pdf        # informe final
    ├── amenazas.xlsx      # tabla de amenazas
    └── img/               # capturas de los DFD y del árbol de ataque
```

## Cómo abrir el modelo

Requisitos: OWASP Threat Dragon 2.6.2, versión de escritorio, instalada desde sus releases en GitHub. Lo probamos en Windows.

1. Descargar o clonar este repositorio.
2. Abrir OWASP Threat Dragon y elegir
 **Open an existing threat model**.
3. Seleccionar el archivo `model/eventosu.json`.
4. En la lista de diagramas, abrir **EventosU - Nivel 0 (contexto)** o **EventosU - Nivel 1**.

## Resumen del contenido

- **DFD:** nivel 0 con EventosU como un solo proceso, y nivel 1 con todos los componentes y cinco límites de confianza.
- **Amenazas:** 18 amenazas identificadas con STRIDE, tres por categoría.
- **Árbol de ataque:** objetivo "entrar a un evento sin pagar", con el camino más barato marcado.
- **Matriz de riesgo:** calificación de impacto × probabilidad, con justificación de cada amenaza.
- **Trazabilidad:** las 5 amenazas de mayor riesgo, ligadas a requisitos de OWASP ASVS 5.0, una tarea y un control de verificación.

## Recursos utilizados

- [OWASP Threat Dragon](https://owasp.org/www-project-threat-dragon/)
- [Threat Modeling Manifesto](https://www.threatmodelingmanifesto.org/)
- [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)