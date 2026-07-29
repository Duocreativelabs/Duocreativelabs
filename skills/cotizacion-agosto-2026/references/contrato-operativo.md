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
Stripe Checkout: ...
Estado: sesión creada; pago pendiente de confirmación.
```

## Checklist de seguridad

- Credenciales de n8n/Stripe/Drive referenciadas por credencial administrada, nunca incrustadas.
- Catálogo y reglas de pago validados en servidor.
- Escape correcto de datos insertados en HTML y TXT.
- Nombres de archivos saneados para impedir rutas arbitrarias.
- Drive restringido y con permisos mínimos.
- Confirmación final de pago mediante webhook firmado de Stripe.
- Pruebas sin borrar workflows, credenciales, ejecuciones ni archivos reales.
