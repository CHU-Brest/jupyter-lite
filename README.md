# JupyterLite — Python (Pyodide) & R (xeus)

Site JupyterLite (interface **JupyterLab**) statique, avec deux kernels :

- **Python (Pyodide)** : `duckdb`, `polars`, `pandas`, `numpy`, etc. disponibles.
  Les paquets sont chargés à la demande depuis le **CDN Pyodide (jsdelivr)** au
  premier `import`.
- **R (xeus-r)** : compilé en WebAssembly et **embarqué dans le site** au build.

## Accès réseau : qui a besoin de quoi ?

| | Au **build** (GitHub Actions) | À l'**exécution** (navigateur) |
|---|---|---|
| **R (xeus-r)** | télécharge depuis emscripten-forge, **bundlé dans `dist/`** | rien (100% offline) |
| **Python (Pyodide)** | rien | télécharge Pyodide + paquets depuis le **CDN jsdelivr** |

> ✅ Le serveur (GitLab interne) n'a **jamais** besoin d'accéder à prefix.dev.
> ⚠️ En revanche, pour le kernel **Python**, le **poste de l'utilisateur** doit
> pouvoir joindre le CDN public `cdn.jsdelivr.net` (mais pas prefix.dev). Le
> kernel **R** fonctionne lui totalement hors-ligne.

## Structure

| Fichier | Rôle |
|---|---|
| `environment-r.yml` | Paquets du kernel R (xeus-r) |
| `jupyter_lite_config.json` | Indique à xeus le fichier d'environnement R |
| `.github/build-environment.yml` | Dépendances de build (xeus + pyodide-kernel) |
| `.github/workflows/deploy.yml` | Workflow GitHub Actions |
| `content/` | Notebooks d'exemple inclus dans le site |

Le kernel **Python (Pyodide)** est fourni par `jupyterlite-pyodide-kernel` ; il
n'a pas de fichier d'environnement (les paquets viennent du CDN). Pour
`duckdb` et `polars`, **installez-les d'abord** dans le notebook :

```python
%pip install duckdb polars
import duckdb, polars as pl
```

> ⚠️ `import duckdb` seul **ne suffit pas** : DuckDB n'est pas auto-importable
> dans Pyodide, il faut `%pip install duckdb` au préalable (piplite récupère le
> *wheel* Pyodide de DuckDB). Voir la
> [démo officielle DuckDB](https://duckdb.org/2024/10/02/pyodide).

## Récupérer les fichiers statiques

Le workflow `Build JupyterLite` se déclenche :

- automatiquement à chaque push sur `main` ;
- manuellement via **Actions → Build JupyterLite → Run workflow**.

Il produit le site dans `dist/` puis :

1. **Pousse le contenu sur la branche `gh-pages`** (réécrite à chaque build).
2. **Publie un artefact `jupyterlite-static`** (zip téléchargeable).

## Héberger sur GitHub Pages (optionnel)

1. **Settings → Pages**
2. **Source** : *Deploy from a branch*
3. **Branch** : `gh-pages` / `(root)` → **Save**

## Importer dans le GitLab interne

1. Récupérez les fichiers (clone de `gh-pages` ou artefact zip).
2. Servez le contenu de façon statique (GitLab Pages, Nginx, etc.).
3. Ouvrez `index.html` (ou `lab/index.html` pour aller directement dans Lab).

> ℹ️ Sous un sous-chemin (ex. `https://gitlab.interne/groupe/projet/`), les
> chemins relatifs de JupyterLite fonctionnent généralement tels quels. Sinon,
> reconstruisez avec `jupyter lite build --base-url /groupe/projet/ …`.

## Personnaliser

- **R** : ajoutez des paquets `r-<nom>` dans `environment-r.yml` (s'ils existent
  sur [emscripten-forge](https://emscripten-forge.org/)).
- **Python** : aucun fichier à modifier — installez à la volée dans un notebook
  via `import` (paquets de la distribution Pyodide) ou
  `import piplite; await piplite.install("mon_paquet")`.

## Build local (optionnel)

```bash
micromamba create -n build-env -f .github/build-environment.yml
micromamba activate build-env
cp README.md content/
jupyter lite build --contents content --output-dir dist
```

> Note : Pyodide n'est **pas** embarqué (mode CDN). Pour un mode 100% offline
> côté Python aussi, il faudrait embarquer la distribution Pyodide
> (`--pyodide pyodide-0.29.3.tar.bz2`), ce qui alourdit fortement la sortie.
