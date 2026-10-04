# Billfolio

Billfolio es una aplicación web de una sola página (HTML + CSS + JavaScript, sin backend propio) para que freelancers y profesionales independientes generen **cotizaciones, contratos de servicios y cuentas de cobro** con su marca, lleven su historial y vean informes de facturación — con los datos guardados de forma segura en la nube y accesibles desde cualquier dispositivo.

## Qué hace

- Genera 3 tipos de documento en PDF, con logo y marca de agua personalizados:
  - **Cotización** — propuesta inicial para el cliente.
  - **Contrato de servicios** — formaliza el acuerdo.
  - **Cuenta de cobro** — documento final para cobrar.
- **Encadena documentos:** una cotización se puede convertir directamente en un contrato o cuenta de cobro, precargando cliente e ítems. Al hacerlo, la cotización queda marcada como "Completada".
- Historial completo por usuario, con edición y anulación de documentos (funciones Pro).
- Informes de facturación con gráfico, filtro de fechas y margen de rentabilidad por documento (función Pro).
- Respaldo exportable/importable en JSON, además del guardado automático en la nube.
- Autenticación de usuarios: enlace mágico por correo o inicio de sesión con Google (Supabase Auth).
- Modelo freemium con límites gratis independientes por tipo de documento (10 cotizaciones, 5 contratos, 5 cuentas de cobro); el desbloqueo completo (Billfolio Pro) se activa manualmente desde Supabase tras confirmar el pago.

## Stack técnico

- **Frontend:** HTML + CSS + JavaScript vanilla, en un único archivo (`index.html`). Sin build step ni framework.
- **PDF:** [jsPDF](https://github.com/parallax/jsPDF) (CDN).
- **Gráficos:** [Chart.js](https://www.chartjs.org/) (CDN).
- **Backend:** [Supabase](https://supabase.com) (Postgres + Auth), vía `@supabase/supabase-js` (CDN). No hay servidor propio.
- **Hosting:** [Vercel](https://vercel.com), desplegado automáticamente desde este repositorio de GitHub.

## Estructura del repositorio

```
index.html   → toda la aplicación (interfaz, lógica y estilos en un solo archivo)
README.md    → este archivo
```

El esquema SQL de la base de datos (tablas, Row Level Security y el trigger de protección de Billfolio Pro) se gestiona directamente en el SQL Editor de Supabase; no es parte del código desplegado.

## Configuración desde cero (Supabase)

1. Crea un proyecto en [supabase.com](https://supabase.com).
2. Ve a **SQL Editor → New query**, pega el esquema SQL del proyecto (crea las tablas `business_profiles` y `documents`, activa Row Level Security con políticas por usuario, y protege la columna `unlocked` con un trigger) y ejecútalo.
3. En **Settings → API**, copia la **Project URL** y la **anon/publishable key**, y actualiza las constantes `SUPABASE_URL` y `SUPABASE_ANON_KEY` al inicio del bloque `<script>` en `index.html`.
4. En **Authentication → URL Configuration**, configura:
   - **Site URL:** la URL donde queda publicada la app (ej. `https://tu-proyecto.vercel.app`).
   - **Redirect URLs:** `https://tu-proyecto.vercel.app/**`.
5. En **Authentication → Providers**:
   - **Email:** activado por defecto (enlace mágico / OTP).
   - **Google:** crea credenciales OAuth de tipo **"Aplicación web"** en Google Cloud Console, con el redirect URI `https://<tu-proyecto>.supabase.co/auth/v1/callback`. Pega el Client ID y Client Secret en el proveedor de Google de Supabase, activa el interruptor y guarda.
6. (Recomendado) Configura un proveedor SMTP propio en **Authentication → SMTP Settings** (por ejemplo, [Resend](https://resend.com)) para no toparte con el límite de envíos de correo del plan gratis de Supabase durante pruebas.

## Despliegue

Este repositorio está conectado a Vercel: cada vez que se sube o actualiza el archivo `index.html` aquí (en minúsculas — el nombre es sensible a mayúsculas), Vercel despliega automáticamente la nueva versión en producción.

## Activar Billfolio Pro para un usuario

El desbloqueo de Pro (documentos ilimitados, edición de documentos, informes) se activa **manualmente**, nunca por el propio usuario:

1. Confirma el pago del usuario (fuera de la app).
2. Ve a **Supabase → Table Editor → `business_profiles`**.
3. Busca su fila (cruza por correo con `auth.users` si hace falta) y marca la columna `unlocked` en `true`.

Un trigger en la base de datos (`protect_unlocked_column`) impide que un usuario autenticado modifique esta columna por su cuenta (por ejemplo, desde la consola del navegador).

## Notas

- Los documentos generados son cotizaciones, contratos de servicios y cuentas de cobro de uso común entre freelancers — **no** son facturas electrónicas con validez tributaria ante la DIAN (Colombia).
- Cada documento y cada perfil de negocio se guardan como un bloque `jsonb` completo en Supabase, lo que permite agregar nuevos campos a la app sin tener que migrar el esquema de la base de datos.
