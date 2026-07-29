---
name: cotizacion-agosto-2026
description: Crea, modifica, publica y valida cotizaciones interactivas de Duo Creative Labs con la plantilla aprobada en agosto de 2026, reutilizando Webhub, Redis y workflows n8n existentes. Úsala cuando se solicite una cotización o quote de cliente, carrito de servicios, checkout Stripe, freeze HTML, resumen TXT o archivo privado en Google Drive.
---

# Cotización Agosto 2026

## Objetivo

Reutilizar el sistema aprobado con Hebron Painting sin inventar otra arquitectura. Crear una cotización bilingüe desde la plantilla incluida, publicarla en una ruta propia de Webhub, conectar Stripe y guardar el freeze HTML y el TXT detallado en las carpetas privadas del cliente.

## Datos obligatorios

Antes de modificar o publicar, obtener o descubrir:

1. Ruta absoluta de la carpeta local del cliente.
2. Nombre oficial, ID de cliente, folio y vigencia.
3. Servicios, precios, idioma y contexto comercial aprobados.
4. Slug Webhub, por ejemplo `cotizacionHebron`.
5. Carpetas restringidas de Google Drive:
   - `Documents/Screenshot HTML`
   - `Documents/TXT`
6. Workflow y endpoints existentes que se reutilizarán.

Si falta la carpeta del cliente o no puede descubrirse con seguridad, detenerse y pedirla. Nunca hacer pública una carpeta de Drive para facilitar el acceso.

## Fuentes y precedencia

1. Instrucciones explícitas del usuario.
2. Precios oficiales vigentes y archivo de cliente.
3. Plantillas de `assets/`.
4. Referencias de esta skill.

No copiar precios antiguos desde otras cotizaciones. No cambiar colores, tipografía, organización o texto no solicitado cuando se esté corrigiendo un documento existente.

## Flujo obligatorio

### 1. Preparar los documentos

- Copiar `assets/Template_Cotizacion_Agosto_2026.html` y `assets/Template_Quote_August_2026.html` a la carpeta del cliente.
- Sustituir datos de Hebron por los del nuevo cliente sin alterar el diseño aprobado.
- Cuando el cliente entregue una imagen de contexto, ubicarla inmediatamente después del bloque `Contexto del Proyecto / Project Context`, centrada y adaptable, sin romper el ancho del documento.
- Mantener HTML y JavaScript sincronizados: nombre, precio, cantidad, cálculo y payload deben coincidir.
- Preservar UTF-8 de extremo a extremo. Rechazar mojibake como `Ã`, `Â` o `ðŸ`.

### 2. Mantener el carrito verificable

- Servicios independientes se acumulan.
- Planes Basic, Pro y Premium son mutuamente excluyentes.
- Google Business Profile es un servicio independiente de pago único por `$300 CAD`, ubicado inmediatamente después de Optimización de Redes Sociales.
- Agrupar Social Media Automation y las dos etapas de Mantención Meta Ads bajo una sola sección visual `MONTHLY RETAINER`; no intercalar servicios mensuales entre los pagos únicos.
- Cada selección visible muestra un ticket morado `SELECCIONADO / SELECTED` y su valor.
- Mostrar siempre un desglose y el total de productos.
- Al deseleccionar, remover el ticket y restar exactamente el valor.

### 3. Aplicar pagos del lado servidor

- Proyectos menores de `$1,000 CAD`: pago total obligatorio.
- Proyectos de `$1,000 CAD` o más: permitir pago total o anticipo fijo de `$1,000 CAD`.
- Si el navegador solicita anticipo para un total menor, normalizar a pago total.
- Stripe debe recibir el mismo catálogo y total que el resumen visible.
- El cliente conserva la opción de pagar el total completo cuando el anticipo sea elegible.

### 4. Reutilizar n8n y Webhub

- Buscar primero en la bitácora y en n8n el workflow operativo del sistema.
- Crear respaldo JSON antes de cualquier edición en vivo.
- No crear, clonar ni desplegar un workflow por cliente si el flujo actual admite una nueva ruta o configuración.
- Publicar el HTML crudo UTF-8 en Redis y servirlo mediante:
  `https://duolabs.fun/webhook/webhub/<slug>`
- Usar un endpoint de checkout existente y parametrizado. No exponer credenciales, API keys ni tokens.
- Validar workflow antes y después; preferir cambios parciales y quirúrgicos.

### 5. Congelar la aceptación

Al crear la sesión de checkout:

- Generar un HTML estático que conserve el diseño.
- Inyectar `PURCHASE SELECTION SUMMARY` con cada servicio, cantidad, valor, total, pago de hoy y saldo.
- Reemplazar controles interactivos por el plan efectivo estático: `PAGO TOTAL (100%)` o `PAGAR ANTICIPO DE $1,000`.
- No afirmar que Stripe confirmó el pago. El freeze registra selección y sesión de checkout; la confirmación definitiva requiere `checkout.session.completed`.
- Guardar en `Screenshot HTML`; no usar una captura PNG salvo que el usuario la pida.

### 6. Crear el TXT

Guardar en `TXT` un archivo UTF-8 siguiendo el formato `FICHA DE CLIENTE INTERNO`:

- cliente, folio, fecha y contactos;
- resumen del proyecto;
- historial de cotizaciones sin borrar entradas anteriores;
- servicios seleccionados y cantidades;
- subtotal, método efectivo, monto de hoy y saldo;
- enlace de Stripe una sola vez, dentro de `DETALLE DE LA SELECCIÓN ACTUAL`;
- estado exacto: sesión creada y pago pendiente de confirmación.

El historial funciona como desglose comercial: cada entrada conserva fecha, folio, servicios, subtotal, modalidad, monto solicitado, saldo y estado. No insertar URLs de Stripe dentro de `HISTORIAL DE COTIZACIONES`, ni repetir allí la sección actual completa.

### 7. Verificar con dos pruebas

Ejecutar pruebas distintas:

1. Total menor de `$1,000` solicitando anticipo: debe forzar 100%.
2. Total mayor o igual a `$1,000` con anticipo: debe cobrar `$1,000` y mostrar saldo.

En ambas comprobar:

- respuesta y ejecución n8n exitosas;
- URL de Stripe bajo `checkout.stripe.com`;
- desglose y plan estáticos correctos en el freeze;
- TXT detallado sin caracteres corruptos;
- archivos en las dos carpetas privadas correctas;
- URL Webhub con HTTP 200 y vista visual, no JSON ni código escapado.

### 8. Conservar el checkpoint de entrega

Considerar la entrega lista únicamente cuando se cumpla la definición de terminado de `references/contrato-operativo.md`. No degradar un elemento ya validado al incorporar correcciones posteriores. Aplicar las observaciones de Claudia de forma quirúrgica, una vez recibidas, y repetir las pruebas afectadas.

## Cierre

Actualizar la memoria del cliente y `BITACORA_GOKU.md`. Informar URL pública de la cotización, rutas locales, carpetas privadas y pruebas realizadas.

Leer `references/contrato-operativo.md` antes de tocar n8n, Stripe o Drive.
