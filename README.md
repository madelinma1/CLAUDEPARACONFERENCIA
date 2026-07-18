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

1. **Desplegar el backend** (Railway/Render/Fly con Postgres + Redis; ver su README). Anotar la URL pública, p. ej. `https://api.madelinsantana.com`. En sus variables de entorno, `PUBLIC_CORS_ORIGINS` debe incluir `https://madelinma1.github.io`.
2. **`signup.html`** — poner la URL del backend en la variable `PLENA_API_URL` (dentro del `<script>` al final del archivo).
3. **Pago con Stripe** — crear un [Payment Link](https://dashboard.stripe.com/payment-links) de $25 con metadata `program=PLENA La Mujer que Emergió`, `cohort=Agosto 2026`, `program_start_date=2026-08-08T13:00:00-04:00`, y ponerlo en la variable `PAY_LINK` de `payment.html` (la página añade `prefilled_email` automáticamente para que el pago se asocie al registro correcto). En el dashboard de Stripe, registrar el webhook `https://<backend>/webhooks/payment` con los eventos `checkout.session.completed`, `charge.refunded` y `charge.dispute.created`.

El registro en la hoja de Google ("PLENA Conferencia - Registros") sigue funcionando en paralelo — el backend no lo reemplaza.
