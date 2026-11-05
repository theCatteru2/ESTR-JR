# JRMarket

JRMarket es una aplicación web experimental vinculada al ecosistema de
JopJoir. Su objetivo es ofrecer una interfaz para consultar activos,
operar con monedas del sistema y administrar mercados sintéticos.

**Estado:** prototipo en desarrollo. No debe considerarse una plataforma
financiera terminada ni un sistema con operaciones garantizadas.

## Qué incluye la interfaz

Según la versión actual del archivo `index.html`, la aplicación contiene
estas secciones y herramientas:

-   **Inicio:** resumen de saldos, activos y datos generales.
-   **Mercados:** listado de tokens sintéticos y acceso a sus vistas de
    operación.
-   **DUL Hub:** cotización calculada de DULUCK, circulación, reserva
    ESTR y operaciones directas de compra y venta.
-   **Intercambio:** formulario para intercambiar activos disponibles.
-   **Predicciones:** mercados YES/NO con un modelo de pool CPMM.
-   **Emisión de sintéticos:** formulario para crear tokens con
    parámetros de curva de vinculación.
-   **Billetera:** saldos, portafolio y movimientos P2P.
-   **Administración:** herramientas para parámetros de DULUCK, gestión
    de mercados y edición del registro de la propia cuenta.
-   **Vista móvil:** navegación inferior con accesos a Inicio, Mercados,
    Intercambiar y Billetera; las demás secciones se agrupan en «Más».

Que una sección o botón exista en la interfaz no significa que todas sus
operaciones estén verificadas o disponibles en la configuración actual
de Firebase.

## Tecnologías y servicios

-   HTML, CSS y JavaScript en una aplicación de página única.
-   Firebase Realtime Database del proyecto de JopJoir para consultar
    cuentas, nombres de usuario e inventarios.
-   Cloud Firestore del proyecto `ranking-a703b` como fuente de lectura
    para determinadas colecciones, entre ellas `markets`,
    `synthetic_tokens` y `system_config`.
-   `localStorage` para conservar una referencia local de la sesión.

No hay un servidor propio incluido en este repositorio. La lógica de la
aplicación se ejecuta principalmente en el navegador.

## Cómo se conecta con JopJoir

El acceso utiliza el sistema existente de JopJoir:

1.  Busca el nombre normalizado en `usernames/{nombre}`.
2.  Compara la contraseña ingresada con el campo `pass` del registro.
3.  Obtiene el UID asociado.
4.  Consulta `users/{uid}` y comprueba que el perfil exista y no esté
    marcado como baneado.

No utiliza Firebase Authentication ni Google Authentication.

La sesión local guarda el UID y el nombre de usuario en `localStorage`.
Al volver a abrir la aplicación, comprueba que el perfil siga existiendo
y no esté baneado. Esta restauración evita volver a ingresar las
credenciales en cada visita, pero **no equivale a una sesión autenticada
y protegida por Firebase Authentication**.

## Arquitectura de datos: limitación importante

La versión actual conserva un puente de compatibilidad con la API de
Realtime Database. En el código:

-   Algunas lecturas de `markets`, `synthetic_tokens` y `system_config`
    se consultan primero en Firestore.
-   Si ciertos documentos no existen, hay rutas que recurren a Realtime
    Database como respaldo.
-   Las escrituras heredadas realizadas mediante `db.ref(...)` se envían
    a Realtime Database.
-   La visibilidad global de Predicciones lee y escribe directamente en
    Realtime Database, en `system_config/app_settings`.

Por lo tanto, **la lectura desde Firestore y la escritura en Realtime
Database no forman una sincronización automática**. Si los datos están
cargados en Firestore pero una operación los modifica en Realtime
Database, ambas fuentes pueden quedar desactualizadas entre sí. Esta
separación debe resolverse antes de considerar confiables las
operaciones sobre mercados y tokens.

## Estado conocido y limitaciones

-   Las operaciones dependen de las reglas y de la estructura real de
    ambas bases de Firebase.
-   Las reglas de Firestore no conceden permisos sobre Realtime
    Database, ni al contrario.
-   El login propio de JopJoir no proporciona por sí solo una identidad
    que las reglas de Firebase puedan verificar mediante `request.auth`.
-   La opción de ocultar Predicciones necesita permiso de escritura
    sobre `system_config/app_settings` en Realtime Database. En la
    configuración probada, esa escritura devuelve `permission_denied`;
    por eso el cambio global no se guarda.
-   No se debe solucionar ese error habilitando escrituras públicas. La
    autorización administrativa actual se comprueba en JavaScript
    mediante un UID, y esa comprobación del navegador no constituye una
    protección de base de datos.
-   Las operaciones de compra, venta, intercambio, predicciones y
    emisión se ejecutan desde el cliente. No hay un backend propio que
    valide y ejecute las transacciones de forma confiable.
-   No se ha verificado que todas las operaciones mantengan balances,
    reservas, circulación e historiales consistentes en escenarios
    simultáneos o ante fallos parciales.
-   Los valores de reserva USD de DULUCK se describen en la interfaz
    como proyección contable. No representan fondos reales ni
    intervienen en el cálculo de la cotización.
-   Los gráficos pueden mostrar datos históricos disponibles; la
    presencia de controles de período o de visualización no garantiza
    que exista historial suficiente para todos los rangos.
-   La interfaz móvil fue adaptada, pero no se certificó su
    comportamiento en todos los tamaños de pantalla ni dispositivos.
-   Abrir el HTML directamente con una dirección `file://` puede
    provocar restricciones de origen del navegador. Para probarlo
    localmente, conviene servir la carpeta mediante un servidor HTTP.

## Ejecución local

Se necesita una conexión a Internet para cargar las bibliotecas y
conectarse a Firebase.

Desde la carpeta que contiene `index.html`, se puede iniciar un servidor
local con Python:

``` bash
python -m http.server 8000
```

Luego abrir:

``` text
http://localhost:8000/
```

Esto evita abrir el documento directamente como `file://`. No instala
dependencias ni configura Firebase: solo sirve los archivos localmente.

## Antes de usarlo con datos reales

1.  Revisar las reglas de Realtime Database y Firestore por separado.
2.  Definir una única fuente de verdad para cada colección.
3.  Corregir la separación entre lecturas de Firestore y escrituras en
    Realtime Database.
4.  Diseñar una autorización administrativa que la base de datos pueda
    verificar; no depender únicamente de una condición JavaScript.
5.  Revisar el manejo de credenciales del login heredado de JopJoir.
6.  Probar cada operación con cuentas y saldos de prueba, incluyendo
    fallos, concurrencia y consistencia de reservas.
7.  Verificar la interfaz en dispositivos móviles reales.

## Alcance

JRMarket es actualmente un prototipo funcional a nivel de interfaz y
lógica de cliente, con integración parcial a las bases de datos
existentes de JopJoir. Algunas funciones están implementadas en el
código, pero la arquitectura de permisos y persistencia todavía presenta
limitaciones conocidas. No se presenta como una plataforma terminada,
auditada ni segura para custodiar activos reales.
