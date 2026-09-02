# Medición de la landing — cómo prenderla

La landing (`index.html`) ya tiene todo el cableado de medición puesto, pero **apagado**.
Está apagado a propósito: cada ID arranca con `XXXX` y, mientras diga `XXXX`, ese bloque
no carga ningún script ni le pega a ningún servidor. Prenderlo es pegar el ID y publicar.
No hace falta tocar nada más.

## Qué se mide

Todas las conversiones de este sitio son **salidas a WhatsApp**. No hay checkout ni
formulario que termine en el sitio, así que el objetivo es siempre el mismo: alguien
abrió el WhatsApp de Advance Group.

Hay dos eventos:

| Evento (GA4)     | Cuándo se dispara                                              | Meta      |
|------------------|----------------------------------------------------------------|-----------|
| `whatsapp_click` | Click en cualquier link a `wa.me`: botón flotante, CTA por unidad de negocio, links sueltos | `Contact` |
| `generate_lead`  | Envío del formulario de cotización (abre WhatsApp con el mensaje armado) | `Lead`    |

`whatsapp_click` viaja con `origen` (`boton_flotante` / `unidad_de_negocio` /
`link_whatsapp`), así se puede ver qué CTA trae más gente. `generate_lead` viaja con
`producto` y `pais_origen` cargados en el formulario.

## Dónde se pegan los IDs

Todo junto en `index.html`, arriba de todo, en el bloque `window.ADV_TRACKING`
(está señalizado con un comentario grande que dice MEDICIÓN):

```js
window.ADV_TRACKING = {
  GA4_ID:    'G-XXXXXXXXXX',
  ADS_ID:    'AW-XXXXXXXXX',
  ADS_LABEL: 'XXXXXXXXXXXXXXXXXX',
  PIXEL_ID:  'XXXXXXXXXXXXXXX'
};
```

- **`GA4_ID`** — Google Analytics ▸ Administrar ▸ Flujos de datos ▸ el `G-…`
- **`ADS_ID`** — Google Ads ▸ Herramientas ▸ Conversiones ▸ el `AW-…`
- **`ADS_LABEL`** — misma pantalla, lo que va **después** de la barra.
  En `AW-123456789/AbC-D_efG` la etiqueta es `AbC-D_efG`.
- **`PIXEL_ID`** — Meta ▸ Administrador de eventos ▸ Orígenes de datos ▸ el número del
  pixel. Son 15 o 16 dígitos, solo números, sin letras ni guiones.

Se pueden prender por separado. Cargar el pixel de Meta no requiere tocar los de Google
ni al revés: cada bloque mira su propio ID.

## El `<noscript>` de Meta

Justo después de `<body>` hay un `<noscript>` del pixel **comentado**. Es para las
visitas sin JavaScript. Cuando cargues `PIXEL_ID`, descomentalo y reemplazá los `XXXX`
de la URL por el mismo número. Si no lo hacés no pasa nada grave — se pierde una franja
chica de tráfico.

## Cómo verificar que quedó andando

1. Publicar (todo lo que entra a `main` se deploya solo por Netlify).
2. Abrir https://advancegrouparg.com con la extensión **Meta Pixel Helper** (Chrome):
   tiene que ver el pixel y un `PageView`.
3. Clickear el botón flotante de WhatsApp → el Helper tiene que mostrar un `Contact`.
4. Mandar el formulario de cotización → un `Lead`.
5. En Meta ▸ Administrador de eventos ▸ **Probar eventos** se ven en vivo, sin esperar.
6. Para Google, lo mismo con GA4 ▸ Informes ▸ **Tiempo real**.

## Lo que falta / lo que sigue

- **CAPI (conversiones desde el servidor).** El pixel del navegador se pierde entre un
  30% y un 50% de los eventos por bloqueadores e iOS. ADVAPP (el CRM) ya tiene la
  variable `META_CRM_DATASET_ID` preparada para mandar las conversiones por API desde el
  backend, pero está vacía. Es el paso que de verdad mejora la atribución, y es un
  trabajo aparte de esta landing.
- **Deduplicación.** Si algún día se prende CAPI *y* el pixel para el mismo evento, hay
  que mandar un `eventID` compartido en los dos lados o Meta cuenta doble.
- **Consentimiento.** Hoy no hay banner de cookies. Para tráfico de la UE haría falta;
  para Argentina, con la política de privacidad que ya está publicada en
  `/privacidad.html`, alcanza.
