# CLAUDEPARACONFERENCIA

Página de la Conferencia PLENA: La Mujer que Emergió — sábado 8 de agosto, 2026, en Heritage Park, 1 Jackson St, Lawrence, MA.

- `index.html` — página principal del evento
- `signup.html` — formulario de reserva (nombre, correo y comentarios opcionales)
- `payment.html` — confirmación y pago ($25 USD)

## Funnel de emails automatizado

La página está integrada con el backend de automatización
[madelinma1/CLAUDEFORCOLLEGE](https://github.com/madelinma1/CLAUDEFORCOLLEGE), que gestiona dos agentes:

1. **Recuperación de registros abandonados** — si alguien deja sus datos en `signup.html` pero no paga, recibe emails de seguimiento generados con IA a los 3 y 5 días (personalizados con sus comentarios). Se cancelan automáticamente al pagar.
2. **Onboarding de clientas pagadas** — al confirmarse el pago, se crea el contacto (sin duplicados), se aplican etiquetas y se envía la secuencia de bienvenida (confirmación inmediata, preparación, recordatorio antes del evento). Se detiene ante reembolsos, disputas o bajas.

### Activación (3 pasos)

La integración está **apagada por defecto** — la página funciona igual que antes hasta completar esto:

1. **Desplegar el backend** (Railway/Render/Fly con Postgres + Redis; ver su README). Anotar la URL pública, p. ej. `https://api.madelinsantana.com`. En sus variables de entorno: `PUBLIC_CORS_ORIGINS` debe incluir `https://madelinma1.github.io`, y definir `WIX_WEBHOOK_SECRET` con un valor secreto largo.
2. **`signup.html`** — poner la URL del backend en la variable `PLENA_API_URL` (dentro del `<script>` al final del archivo).
3. **Wix Automations** — en el sitio de Wix donde vive el paylink, crear una automatización: *disparador* "Se pagó un pedido/factura" → *acción* "Enviar solicitud HTTP (webhook)" con:
   - **URL:** `https://<backend>/webhooks/wix/payment?secret=<WIX_WEBHOOK_SECRET>` (o el header `x-webhook-secret` si tu plan lo permite)
   - **Método:** POST, cuerpo JSON:
     ```json
     {
       "email": "{{email del comprador}}",
       "firstName": "{{nombre}}",
       "lastName": "{{apellido}}",
       "transactionId": "{{número de pedido}}"
     }
     ```
     Solo `email` es obligatorio — el programa, la cohorte y la fecha de inicio se completan con los valores configurados en el backend (`PROGRAM_NAME`, `PROGRAM_COHORT_START_DATE`).
   - Si tu plan tiene disparador de reembolso, crear otra automatización igual hacia `/webhooks/wix/refund`. Si no, usar `POST /admin/contacts/cancel` del backend cuando haya un reembolso.

**Importante:** la automatización asocia el pago con el registro **por correo electrónico**. La página de pago le recuerda a la compradora usar el mismo correo con el que se registró. Para ventas en puerta o pagos fuera de Wix, el backend tiene `POST /admin/payments/confirm`.

El registro en la hoja de Google ("PLENA Conferencia - Registros") sigue funcionando en paralelo — el backend no lo reemplaza.
