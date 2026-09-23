# Finanzas: instalación

La app tiene dos partes:

- **Backend (Google Apps Script):** `Code.gs` y `appsscript.json`. Corre dentro de tu cuenta Google, lee los correos de los bancos y guarda todo en una hoja de cálculo tuya.
- **App (web instalable, funciona sin internet):** `index.html`, `sw.js`, `manifest.json`, `icon-192.png` e `icon-512.png`. Tus datos quedan guardados en cada dispositivo y se sincronizan con el backend solo cuando tocas **Sincronizar**.

Tiempo estimado: unos 30 minutos la primera vez.

---

## 1. Backend en Apps Script

1. Entra a [script.google.com](https://script.google.com) con la cuenta de Gmail donde llegan los vouchers y crea un **Proyecto nuevo**. Ponle de nombre «Finanzas».
2. Borra el contenido de `Código.gs` y pega el contenido completo de `Code.gs`.
3. Ve a **Configuración del proyecto** (ícono de engranaje), activa **Mostrar el archivo de manifiesto "appsscript.json"** y reemplaza ese archivo por el `appsscript.json` incluido. Esto fija la zona horaria en Santiago.
4. Vuelve al editor, elige la función `configurar` y presiona **Ejecutar**. Google pedirá permisos para:
   - leer Gmail (solo se leen los remitentes de los bancos que actives y los correos con la etiqueta «Finanzas»),
   - crear la hoja de cálculo,
   - consultar mindicador.cl (dólar y UF).

   Como es un script tuyo, Google mostrará «Google no verificó esta app»: entra a **Configuración avanzada → Ir a Finanzas**.
5. Abre **Ejecuciones** (o el registro) y copia el **TOKEN** que aparece. Trátalo como una contraseña.
6. **Implementar → Nueva implementación → Tipo: Aplicación web**
   - Ejecutar como: **Yo**
   - Quién tiene acceso: **Cualquier usuario**

   Copia la dirección que termina en `/exec`.

> «Cualquier usuario» es necesario para que la app se conecte sin iniciar sesión en Google en cada dispositivo. Sin el token, el script no entrega ni recibe datos. Si crees que el token se filtró, ejecuta `rotarToken` y pega el nuevo en la app.

Cuando cambies `Code.gs` más adelante, usa **Implementar → Gestionar implementaciones → Editar → Versión: nueva** para que la dirección `/exec` siga siendo la misma.

## 2. Publicar la app

La app necesita estar en una dirección `https` para funcionar sin internet y poder instalarse. La opción más simple y gratuita es GitHub Pages:

1. Crea un repositorio en GitHub (por ejemplo `finanzas`).
2. Sube `index.html`, `sw.js`, `manifest.json`, `icon-192.png` e `icon-512.png`. **No subas** `Code.gs` ni ningún respaldo `.json` con tus datos.
3. **Settings → Pages → Branch: main / root → Save.** En un minuto tendrás una dirección como `https://tuusuario.github.io/finanzas/`.

El código de la app no contiene datos ni el token: esos quedan solo en tus dispositivos. Por eso el repositorio puede ser público. Alternativa: Netlify Drop, arrastrando la carpeta.

## 3. Instalar en el iPhone y en el Mac

**iPhone o iPad:** abre la dirección en Safari → Compartir → **Agregar a pantalla de inicio**.

**Mac (macOS Sonoma 14 o posterior):** abre la dirección en Safari → **Archivo → Agregar al Dock**.
- Queda como una app propia: tiene ícono en el Dock y en Launchpad, abre en su propia ventana y funciona sin internet.
- En pantallas grandes cambia a diseño de escritorio: barra lateral, ingresos y egresos lado a lado, y resumen en dos columnas.
- También puedes instalarla desde Chrome con el ícono de instalar de la barra de direcciones.

**Conectar el primer dispositivo:** Ajustes → pega la dirección `/exec` y el token → **Probar conexión** → marca tus bancos → **Sincronizar**.

**Conectar los demás dispositivos:**
1. En el dispositivo ya configurado, toca **Copiar código para otro dispositivo**.
2. En el nuevo, pégalo en **¿Ya lo configuraste en otro dispositivo?**
3. Los bancos, tus nombres, los nombres de tarjetas y las reglas se copian solos.

Hay un solo archivo `index.html` para el iPhone y el Mac. Al subir la versión nueva a GitHub, ambas apps se actualizan solas la próxima vez que las abras.

**Cómo se mantienen iguales el Mac y el iPhone:** los dos sincronizan contra la misma hoja de Google.
- La app sincroniza sola al abrirla, al recuperar internet y cada 15 minutos mientras está abierta.
- Lo que editas se sube unos segundos después.
- Sin internet, todo sigue funcionando y se sube cuando vuelve la conexión.

**Atajos en el Mac:**

| Atajo | Acción |
|---|---|
| ⌘R | Sincronizar |
| ⌘N | Nuevo movimiento |
| ⌘1 a ⌘6 | Cambiar de sección (⌘4 es Deudas) |
| ⌘← y ⌘→ | Cambiar de mes |

> ¿Por qué no una app nativa de Mac? Sería otra app que mantener aparte y no agregaría funciones: la versión instalada desde Safari ya funciona sin internet, tiene su propio ícono y se actualiza sola cuando actualizas los archivos en GitHub.

La primera sincronización revisa los últimos 90 días (lo puedes cambiar en Ajustes). Si hay muchos correos puede quedar a medias; la app te avisa y basta con sincronizar otra vez. No se duplica nada.

## 4. Correos de Outlook

Apps Script solo lee Gmail. Para que los vouchers que llegan a Outlook entren a la app:

1. En Outlook: **Configuración → Correo → Reglas → Agregar regla**.
2. Condición: el remitente contiene `bancochile.cl`, `santander.cl`, `bancoestado.cl` (y los de tus otros bancos).
3. Acción: **Redirigir a** tu Gmail. Usa «Redirigir», no «Reenviar»: así el correo conserva el remitente original del banco y la app lo reconoce.

Si la cuenta de Outlook es institucional, es posible que la organización bloquee la redirección a correos externos. En ese caso, usa la importación de cartola para esos movimientos.

Opcional: en Gmail, crea la etiqueta **Finanzas** y aplícala con un filtro a cualquier otro remitente que quieras incluir.

## 5. Cartola

Descarga la cartola en Excel o CSV desde el sitio de tu banco. En **Ajustes → Importar cartola**, elige el banco y el archivo. La primera vez confirmas qué columna es la fecha, la descripción, los cargos y los abonos; la app lo recuerda para las siguientes.

La app no duplica movimientos:

- Los que ya llegaron por correo se ocultan como duplicados (mismo monto, sentido y fecha ±3 días; ±6 días para ingresos).
- Volver a importar la misma cartola no agrega nada.

## 6. Deudas

La pestaña **Deudas** junta dos cosas: las compras en cuotas de tus tarjetas y los créditos que ingreses a mano (consumo, hipotecario en UF, préstamos).

Para cada deuda muestra:
- **cuántas cuotas van pagadas** y cuántas faltan, con una barra de avance,
- cuánto queda por pagar,
- el valor de la cuota,
- el mes en que terminas.

Arriba verás la **deuda total**, el **próximo pago** y los **pagos de los próximos 6 meses**.

**Cómo se estiman las cuotas pagadas (por fechas):**
- En el estado de cuenta, una compra «05/06» significa que ese mes se cobra la cuota 5.
- Se cuentan 4 cuotas pagadas antes del estado de cuenta.
- Cuando pasa la fecha de «Pagar hasta», se cuenta la quinta.
- Cada mes siguiente, al pasar la misma fecha, se suma una más.
- Las compras «00/03» (primera cuota el mes siguiente) parten un mes después.

**Créditos ingresados a mano:** indicas el total de cuotas, las ya pagadas y el día de pago. La app suma una cuota pagada cada vez que pasa ese día.

La estimación supone que pagas cada estado de cuenta a tiempo. Sube el estado de cuenta nuevo cada mes: se usa el más reciente de cada tarjeta y corrige cualquier diferencia.

**Subir un estado de cuenta:** Deudas → **Subir estado de cuenta**. Acepta PDF, Excel o CSV.
- **PDF con clave:** la app la pide en el momento y no la guarda.
- Lee el **estado nacional** (en pesos) y el **internacional** (en dólares) del mismo PDF.
- Antes de guardar revisas los cargos, la fecha de pago y la tarjeta.
- **Las suscripciones se excluyen**, porque ya aparecen en su pestaña. Puedes corregirlo cargo por cargo.

Estas cifras son una proyección y **no suman al flujo**: cada compra ya se registró cuando la hiciste. Tampoco se descuentan los saldos a favor del período anterior.

**Probado con un estado de cuenta real de Banco de Chile:**
- Los totales de compras en cuotas, cargos del mes y compras en dólares coinciden exactamente con los del documento.
- La proyección de los próximos 4 meses difiere de la tabla «Vencimiento próximos 4 meses» del banco en menos de $25 por mes. La diferencia se debe al redondeo de la última cuota.

## 7. Datos de tarjeta

**La app no guarda números de tarjeta ni de cuenta.**
- Cada tarjeta aparece solo como «Tarjeta 1 Banco de Chile», «Tarjeta 2 Banco de Chile», etc. Puedes cambiarles el nombre en Ajustes o en la pestaña Tarjetas.
- Los números enmascarados (****1234, «terminada en 1234») se eliminan de los asuntos de correo y de las descripciones, tanto en la app como en la hoja de Google.
- De los estados de cuenta solo se guardan fechas, comercios, montos y cuotas. El nombre del titular, el correo, el número de tarjeta y la clave del PDF nunca se leen ni se guardan.

**Si ya habías instalado la versión anterior:**
1. Pega el `Code.gs` nuevo.
2. Crea una nueva versión de la implementación.
3. Ejecuta una vez `limpiarDatosTarjeta` para borrar los dígitos guardados antes.

La app también los elimina de cada dispositivo al abrirse.

## 8. Cómo cuenta el dinero

- **Suman al flujo:** ingresos, transferencias recibidas, gastos, transferencias enviadas, y descuentos o retenciones.
- **No suman al flujo:**
  - Transferencias entre tus cuentas. Se reconocen con tus nombres y RUT en Ajustes.
  - Pagos de tarjeta de crédito, porque cada compra ya se contó.
  - Duplicados.
- **Sueldo:** si ingresas el líquido y el bruto, el ingreso es el bruto y la diferencia aparece como «Descuentos legales del sueldo» en egresos. El flujo neto queda igual al líquido.
- **Boleta de honorarios:** marca la casilla y registra el bruto. La retención se calcula según el año: 15,25% en 2026, 16% en 2027 y 17% desde 2028.
- **Si ya registraste el sueldo a mano** y después llega por correo la transferencia del mismo monto líquido, la transferencia se oculta como duplicado.

## 9. Si un correo no se reconoce

Cada banco usa plantillas distintas y las cambia de vez en cuando. En **Ajustes → Correos no reconocidos** verás los que no se pudieron leer.

En Apps Script, la función `probarLector` muestra en el registro qué entiende de tus últimos correos bancarios, sin guardar nada. Con dos o tres ejemplos reales por banco (puedes borrar montos y nombres) se ajustan los patrones de `parseEmail` en `Code.gs`.

## Archivos

| Archivo | Dónde va |
|---|---|
| `Code.gs`, `appsscript.json` | Proyecto de Apps Script |
| `index.html`, `sw.js`, `manifest.json`, `icon-192.png`, `icon-512.png` | GitHub Pages u otro hosting `https` |

