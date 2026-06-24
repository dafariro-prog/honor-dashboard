# Tracking de Campañas — HONOR

Dashboard standalone de tracking diario de campañas de pauta de **Honor** (multi-país:
Costa Rica, El Salvador, Guatemala, Honduras, Nicaragua, Panamá, República Dominicana),
sobre Meta, Google Ads, DV360 y TikTok. Cliente de Grupo Garnier.

- **Frontend:** `index.html` estático + Chart.js (sin build).
- **Datos:** `data/tracking.json`, generado desde la vista
  `garnier-436600.Garnier.vw_marketing_daily_campaign` (BigQuery), **filtrando solo las
  cuentas de Honor** y cruzado con `data/homologacion.json` (`platform + account_name`).
- **País corregido:** en la homologación original todas las cuentas Honor estaban mal
  etiquetadas como Panamá; aquí el país se recalcula desde el `account_name`
  (`_CR`→Costa Rica, `_SV`→El Salvador, `_GT`→Guatemala, `_HN`→Honduras, `_NI`→Nicaragua,
  `_PA`→Panamá, `_RD`→República Dominicana).
- **Ventana:** todo el año **2026** (no las ventanas de 90/60 días del dashboard general).
- **Auto-actualización:** GitHub Actions corre `refresh.js` cada día y commitea la data;
  Vercel redeploya en cada push. Funciona aunque tu PC esté apagado.

## Ver en local
```bash
npm run serve      # http://localhost:4500
```

## Refrescar la data manualmente
Requiere una Service Account de GCP con acceso a BigQuery:
```bash
GOOGLE_APPLICATION_CREDENTIALS=./sa.json npm run refresh
```

## Puesta en marcha del auto-refresco (una sola vez)

1. **Service Account en GCP** (proyecto `garnier-436600`):
   - IAM & Admin → Service Accounts → crear una (ej. `honor-tracking-bq`).
   - Roles: **BigQuery Data Viewer** + **BigQuery Job User**.
   - Crear una **clave JSON** y descargarla.
2. **Secret en GitHub:** repo → Settings → Secrets and variables → Actions → New secret
   - Nombre: `GCP_SA_KEY` · Valor: el contenido completo del JSON.
3. **Vercel:** New Project → importar este repo de GitHub → Deploy (framework: *Other*, sin build).
4. Listo: `.github/workflows/refresh.yml` corre a las 11:00 UTC (05:00 CR/PA) y también se
   puede lanzar a mano desde la pestaña **Actions** → *Run workflow*.

> El repo ya incluye un `data/tracking.json` inicial con datos reales de 2026, así que el
> dashboard se ve poblado desde el primer deploy; el cron lo mantiene al día.

## Cuentas incluidas (22)
Definidas en `data/homologacion.json` (`platform||account_name`). `refresh.js` deriva de ahí
la lista de `account_name` para el filtro `IN (...)` de BigQuery.

| Plataforma | Cuentas |
|---|---|
| Meta | Honor_OMD_{COSTA RICA, EL SALVADOR, GUATEMALA, HONDURAS, NICARAGUA, PANAMA, REPUBLICA DOMINICANA} |
| Google Ads | OMD_Honor Mobile_{CostaRica, ElSalvador, Guatemala}, OMD_HONOR Mobile_Panama |
| TikTok | [OMD] HONOR_{CR, GT, PA, RD} |
| DV360 | [OMD] HONOR_{CR, GT, HN, NI, PA, RD, SV} |

## Acceso (Supabase)
Reutiliza el proyecto Supabase existente (`ioumqovyirtwqjrbseqt`) y su tabla `permisos`.
Para dar acceso a alguien al dashboard de Honor, entra a `/admin` (como administrador) y
crea el usuario con rol *admin*, scope vacío, o marcando la "agencia" **Honor**. Los usuarios
con scope de otras agencias del dashboard general no verán datos aquí (este dashboard solo
carga cuentas de Honor).

## Notas
- Gasto en **moneda nativa** (Honor es todo USD; no se convierte).
- Estado de campaña: **>5 días sin consumo = Finalizada**, si no Activa.
- El agente de recomendaciones (Gemini) no está activado en este dashboard; se puede sumar
  después copiando `analyze.js` + `.github/workflows/analyze.yml` del proyecto `tracking_campanas`.
