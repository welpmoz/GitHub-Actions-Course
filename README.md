# GitHub-Actions-Course
Repor for GitHub Actions

## Flujo de desarrollo

### Ramas

| Rama | Propósito | Push directo |
|---|---|---|
| `main` | Producción. Cada push despliega en `PRODUCTION`. | No (solo vía PR) |
| `dev` | Integración. Los PRs `dev → main` despliegan en `STAGING`. | No (solo vía PR) |
| `welpmoz-*` | Ramas de trabajo. | Sí |

El ruleset del repositorio protege **todas** las ramas excepto `welpmoz-*`. Aunque el workflow escucha pushes a `feature-*`, esas ramas también están protegidas, así que crea tus ramas de trabajo con el prefijo `welpmoz-`.

### Pasos

1. Crea tu rama desde `dev` y súbela:
   ```bash
   git fetch origin
   git switch -c welpmoz-<tema> origin/dev
   git push -u origin welpmoz-<tema>
   ```
2. Abre un PR hacia `dev`. El workflow `my-first-workflow` ejecuta `JOB1`, `JOB2` y `JOB3`.
   - `JOB1` y `JOB2` son checks obligatorios.
   - Tu rama debe estar al día con `dev` antes de mergear.
3. Mergea el PR (no requiere aprobaciones).
4. Abre un PR `dev → main`. Además de los checks, el job `STAGE DEPLOYMENT` despliega en `STAGING`.
5. Mergea el PR. El push a `main` dispara `PRODUCTION DEPLOYMENT`, que:
   - queda en espera hasta que @welpmoz lo apruebe (Actions → el run → **Review deployments**);
   - después espera 1 minuto y despliega.

### Environments

| Environment | Ramas permitidas | Aprobación | Variables | Secretos |
|---|---|---|---|---|
| `STAGING` | `dev` | No | `HOST`, `USER` | `PASSWORD` |
| `PRODUCTION` | `main` | @welpmoz + espera de 1 min | `HOST`, `USER` | `PASSWORD` |

Los jobs leen estos valores con `${{ vars.HOST }}`, `${{ vars.USER }}` y `${{ secrets.PASSWORD }}`. El valor depende del environment del job.

### Reglas de las ramas protegidas

- No se permite push directo; todo cambio entra por PR.
- No se permite force-push ni borrar la rama.
- Los checks `JOB1` y `JOB2` deben pasar y la rama debe estar actualizada con la de destino.

### Nota sobre condiciones en el workflow

- `github.ref` es la referencia que disparó el workflow, por ejemplo `refs/heads/main` en un push. Úsala para decidir despliegues.
- `github.head_ref` es la rama **origen** de un PR. Solo existe en eventos `pull_request`; en un `push` está vacía.
- `github.base_ref` es la rama **destino** de un PR.
