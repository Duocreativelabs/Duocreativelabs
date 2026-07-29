# Contrato operativo de Cotización Agosto 2026

## Sistema de referencia validado

- Plantilla visual: cotización y quote de Hebron Painting aprobados el 29 de julio de 2026.
- Publicación: Webhub lee HTML crudo UTF-8 desde una clave Redis.
- Automatización: un workflow existente recibe selección, normaliza catálogo y pago, crea Checkout en Stripe, produce freeze/TXT y sube ambos a Drive.
- Persistencia: `Screenshot HTML` contiene el HTML congelado; `TXT` contiene la ficha detallada.

## Configuración por cliente

Mantener fuera del HTML público:

- IDs de carpetas de Drive;
- identificadores internos de workflow;
- credenciales y tokens;
- datos que no necesita el navegador.

El payload mínimo debe identificar cliente, folio, slug, idioma, selecciones y método solicitado. El servidor recalcula precio, método efectivo, pago de hoy y saldo.

## Invariantes

- No confiar en precios enviados por el navegador.
- Mantener `gbp` como servicio de pago único por `$300 CAD`, separado de la optimización de redes sociales.
- No aceptar dos planes mensuales simultáneos.
- No aceptar anticipo si el subtotal es menor de `$1,000 CAD`.
- No llamar “pagado” a una sesión de Checkout creada.
- No guardar HTML con `\n` literales ni dentro de un objeto JSON visible.
- No publicar carpetas de Drive; compartir solamente con las cuentas autorizadas.
- No modificar un workflow sin respaldo previo.

## Esquema del freeze

La sección estática debe contener:

1. Nombre de cada producto.
2. Cantidad.
3. Precio o importe.
4. Total de productos seleccionados.
5. Pago de hoy.
6. Saldo pendiente.
7. Método efectivo.
8. Estado pendiente de confirmación en Stripe.

## Esquema del TXT

```text
========================================
FICHA DE CLIENTE INTERNO: <ID>
========================================

NOMBRE DEL CLIENTE / EMPRESA
----------------------------------------
<cliente>

CONTACTOS
----------------------------------------
<contactos o no especificados>

RESUMEN DEL PROYECTO
----------------------------------------
¿Qué hace?: ...
¿Qué necesita?: ...
¿Qué ofrecemos?: ...

HISTORIAL DE COTIZACIONES
----------------------------------------
[fecha] <folio>
Servicios seleccionados: ...
Subtotal: ...
Método efectivo: ...
Pago de hoy: ...
Saldo: ...
Estado: sesión creada; pago pendiente de confirmación.
```

El enlace `Stripe Checkout` debe aparecer una sola vez en `DETALLE DE LA SELECCIÓN ACTUAL`, antes del historial. El historial conserva el desglose y los montos de cada cotización, pero nunca repite enlaces de checkout.

## Checklist de seguridad

- Credenciales de n8n/Stripe/Drive referenciadas por credencial administrada, nunca incrustadas.
- Catálogo y reglas de pago validados en servidor.
- Escape correcto de datos insertados en HTML y TXT.
- Nombres de archivos saneados para impedir rutas arbitrarias.
- Drive restringido y con permisos mínimos.
- Confirmación final de pago mediante webhook firmado de Stripe.
- Pruebas sin borrar workflows, credenciales, ejecuciones ni archivos reales.

## Definición de terminado

Este es el checkpoint canónico aprobado:

- HTML visual servido por Webhub, no JSON ni código escapado.
- Google Business Profile aparece después de Optimización de Redes Sociales, con desglose propio y precio de `$300 CAD`.
- Carrito acumulativo y plan mensual mutuamente excluyente.
- Ticket visible en cada servicio seleccionado y total correcto.
- Pago total obligatorio bajo `$1,000 CAD`; desde `$1,000 CAD`, pago completo o anticipo fijo de `$1,000 CAD`.
- Freeze HTML estático con servicios, total, pago de hoy, saldo y modalidad efectiva.
- TXT detallado sin mojibake.
- Enlace de Stripe una sola vez en `DETALLE DE LA SELECCIÓN ACTUAL`.
- Historial con servicios, subtotal, modalidad, pago solicitado, saldo y estado, sin repetir el checkout.
- TXT y freeze guardados en las carpetas privadas correctas.
- Workflow existente reutilizado, activo y con ejecución exitosa.

Prueba de referencia aprobada:

- Ejecución n8n `23803`.
- Setup `$500` + Facebook/Instagram `$200` + Plan Pro `$700`.
- Subtotal `$1,400 CAD`, pago completo.
- TXT y HTML subidos con éxito.
- Enlaces Stripe en TXT: `1`; enlaces Stripe dentro del historial: `0`.
- Freeze con `$1,400 CAD` y `PAGO TOTAL (100%)`.

Corrección de Claudia incorporada:

- Servicio `gbp`: personalización, modificación y upgrade de Google Business Profile.
- Precio: `$300 CAD`, pago único.
- Alcance: auditoría, información comercial, servicios/categorías, identidad visual, optimización local, reseñas/comunicación, configuración técnica y upgrade general.
- Ejecución n8n `23812`: subtotal `$300`, solicitud de anticipo normalizada a pago completo, TXT/HTML subidos, un solo enlace Stripe y freeze estático correcto.

No asumir otras correcciones de Claudia; aplicarlas cuando el usuario entregue la lista.
