# Self-hosting de los widgets del perfil

Las instancias públicas de estos servicios (Vercel/Heroku compartidos) se caen
seguido por rate-limit de la API de GitHub o porque el proveedor las deshabilita.
Este documento deja lo importante bajo control propio.

## Estado tras la migración

| Widget                    | Estrategia                                                           | Runtime       | Acción requerida              |
| ------------------------- | -------------------------------------------------------------------- | ------------- | ----------------------------- |
| Header / footer (capsule) | SVG estático en `assets/`                                            | Ninguno       | ✅ Hecho                      |
| Snake (contribuciones)    | GitHub Action → rama `output`                                        | Ninguno       | ⚙️ Correr el workflow una vez |
| Stats + lenguajes         | Vercel propio → `github-readme-stats-nine-nu-63.vercel.app`          | Vercel (tuyo) | ✅ Hecho (env `PAT_1`)        |
| Actividad reciente        | Vercel propio → `github-readme-activity-graph-rust-sigma.vercel.app` | Vercel (tuyo) | ✅ Hecho (env `TOKEN`)        |
| Trofeos                   | Eliminado                                                            | —             | —                             |
| Streak                    | Instancia pública (`github-readme-streak-stats-eight.vercel.app`)    | Terceros      | Pendiente si vuelve a caerse  |
| Typing SVG                | Instancia pública (`readme-typing-svg.herokuapp.com`)                | Terceros      | Pendiente si vuelve a caerse  |

> **¿Por qué stats y actividad sí son runtime?** Ninguno de los dos tiene una
> GitHub Action que genere un SVG commiteado con colores custom. Pero en tu
> instancia propia, con tu PAT, el rate limit es de 5000 req/h **por token** —
> no se agota como los mirrors públicos compartidos. El problema nunca fue el
> concepto, fue depender de una instancia ajena sin token.

## 1. Personal Access Token (una sola vez)

Crear un **PAT classic** en <https://github.com/settings/tokens/new>:

- Nota: `perfil-widgets`
- Expiración: sin vencimiento (o 1 año y renovar)
- Scopes: `repo`, `read:user`, `read:org`

Copiar el token. Se usa en el paso 2 y en el paso 4 (el mismo token sirve para ambos).

## 2. Stats + lenguajes — github-readme-stats en Vercel

1. Fork de <https://github.com/anuraghazra/github-readme-stats> a tu cuenta.
2. En <https://vercel.com> → **Add New → Project** → importar ese fork.
   Framework preset: **Other**. Deploy.
3. Proyecto → **Settings → Environment Variables**:
   - Name: `PAT_1`
   - Value: el PAT del paso 1
4. **Deployments → ⋯ → Redeploy** para que tome la variable.
5. Copiar el dominio final (ej. `github-readme-stats-jandrescodes.vercel.app`)
   y reemplazar **`TU-DOMINIO-STATS`** (2 ocurrencias) en `README.md`, sección
   _Estadísticas de GitHub_, y descomentar el bloque.

Los parámetros de color en el README ya están alineados con la tarjeta de streak
(`bg_color=1a1a2e`, `title_color=9d8fd1`, `icon_color=777BB4`, `border_color=4b3f72`).

Verificación:

```
curl -s -o /dev/null -w "%{http_code}\n" \
  "https://TU-DOMINIO-STATS/api?username=jandrescodes"
```

Debe devolver `200`.

## 3. Snake

**Actions → Snake animation → Run workflow** (o simplemente pushear: se dispara
solo en push a `main`). No necesita PAT. Crea la rama `output` con
`github-contribution-grid-snake-dark.svg`. El README ya apunta ahí.

## 4. Actividad reciente — github-readme-activity-graph en Vercel

1. Fork de <https://github.com/Ashutosh00710/github-readme-activity-graph> a tu cuenta.
2. En <https://vercel.com> → **Add New → Project** → importar ese fork.
   Framework preset: **Other**. Deploy.
3. Proyecto → **Settings → Environment Variables**:
   - Name: `TOKEN`
   - Value: el PAT del paso 1
4. **Deployments → ⋯ → Redeploy** para que tome la variable.
5. Copiar el dominio final y reemplazar **`TU-DOMINIO-VERCEL`** en `README.md`
   (sección _Actividad Reciente_), y descomentar el bloque.

Verificación:

```
curl -s -o /dev/null -w "%{http_code}\n" \
  "https://TU-DOMINIO-VERCEL/graph?username=jandrescodes&theme=react-dark"
```

Debe devolver `200`.

## 5. Opcional — streak y typing SVG

Hoy funcionan con instancias públicas. Si se caen, mismo patrón:

- **Streak**: fork de `DenverCoder1/github-readme-streak-stats` → Vercel → sin env
  obligatoria (opcional `PATOKEN` para repos privados).
- **Typing SVG**: fork de `DenverCoder1/readme-typing-svg` → Vercel → sin env.

## Mantenimiento

- Si el PAT classic tiene vencimiento, renovarlo antes de que expire (calendario).
  Con "sin vencimiento" no aplica.
- El workflow de la snake corre solo cada día a las 03:00 UTC.
- Los proyectos Vercel se redeployan solos con cada push al fork; no hay que tocarlos.
