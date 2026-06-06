# JupyterLite — Python (Pyodide) avec DuckDB & Polars

Site JupyterLite (interface **JupyterLab**) statique, avec un kernel **Python
(Pyodide)** incluant **DuckDB**, **Polars**, **pandas**, **numpy** et
**pyarrow**.

> 🔑 **Pourquoi Pyodide 0.27 ?** DuckDB et Polars ne sont disponibles que dans la
> distribution **Pyodide 0.27** (ils ont été retirés à partir de Pyodide 0.28
> pour des problèmes de build). La pile est donc épinglée :
> `jupyterlite-pyodide-kernel 0.6.x` ↔ `jupyterlite-core 0.6.x` ↔ Pyodide 0.27.6.

> ℹ️ **R n'est pas inclus.** Le kernel R (xeus-r) exige `jupyterlite-core ≥ 0.7`,
> incompatible avec la pile Pyodide 0.27 nécessaire à DuckDB/Polars. Les deux ne
> peuvent pas cohabiter dans un même site. (Possible via deux sites séparés si
> besoin — demandez.)

## Accès réseau

| | Au **build** (GitHub Actions) | À l'**exécution** (navigateur) |
|---|---|---|
| **Python (Pyodide)** | rien à télécharger | Pyodide + paquets depuis le **CDN jsdelivr** |

> Le serveur (GitLab interne) sert seulement des fichiers statiques. En
> revanche, le **poste de l'utilisateur** doit pouvoir joindre le CDN public
> `cdn.jsdelivr.net` (pour charger Pyodide 0.27, DuckDB, Polars…).

## Utiliser DuckDB / Polars

Dans un notebook (kernel **Python (Pyodide)**) :

```python
%pip install duckdb polars
import duckdb, polars as pl
```

Voir le notebook d'exemple `content/demo-python-duckdb-polars.ipynb`.

## Structure

| Fichier | Rôle |
|---|---|
| `.github/build-environment.yml` | Dépendances de build (pyodide-kernel 0.6.x) |
| `.github/workflows/deploy.yml` | Workflow GitHub Actions |
| `content/` | Notebooks d'exemple inclus dans le site |

Le kernel Python est fourni par `jupyterlite-pyodide-kernel` ; les paquets
viennent du CDN Pyodide, il n'y a donc aucun fichier d'environnement à gérer.

## Récupérer / déployer

Le workflow `Build JupyterLite` (push sur `main` ou *Run workflow* manuel) :

1. construit le site dans `dist/` ;
2. **pousse le contenu sur la branche `gh-pages`** ;
3. **publie un artefact `jupyterlite-static`** (zip téléchargeable).

**GitHub Pages** : Settings → Pages → *Deploy from a branch* → `gh-pages` / `(root)`.

**GitLab interne** : clonez `gh-pages` (ou téléchargez l'artefact) et servez les
fichiers en statique. Ouvrez `index.html` (ou `lab/index.html`).

## Build local (optionnel)

```bash
micromamba create -n build-env -f .github/build-environment.yml
micromamba activate build-env
cp README.md content/
jupyter lite build --contents content --output-dir dist
```
