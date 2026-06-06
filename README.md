# JupyterLite — kernels xeus Python & R (offline)

Site JupyterLite (interface **JupyterLab**) entièrement statique, construit avec
[`jupyterlite-xeus`](https://jupyterlite-xeus.readthedocs.io/). Deux kernels
compilés en WebAssembly sont fournis :

- **Python (xeus-python)** avec `python-duckdb` (module `duckdb`), `pandas`,
  `numpy`, `matplotlib`, `ipywidgets`.
- **R (xeus-r)**.

> ⚠️ **polars** n'est pas inclus : le projet Polars ne se compile pas encore en
> WebAssembly / Emscripten (`polars-runtime-32` n'existe pas sur
> emscripten-forge). **DuckDB** + **pandas** servent d'alternative côté
> navigateur. Voir [pola-rs/polars#19211](https://github.com/pola-rs/polars/issues/19211).

> 💡 **Tout fonctionne hors-ligne.** Au moment du build, tous les paquets conda
> WebAssembly sont téléchargés depuis emscripten-forge **et empaquetés dans le
> dossier `dist/`**. À l'exécution, le navigateur charge ces paquets depuis le
> même hébergeur que le site : **aucun accès à `prefix.dev` n'est nécessaire**.
> Les fichiers générés peuvent donc être servis depuis un GitLab interne isolé.

## Structure

| Fichier | Rôle |
|---|---|
| `environment-python.yml` | Paquets du kernel Python (python-duckdb, pandas, …) |
| `environment-r.yml` | Paquets du kernel R |
| `jupyter_lite_config.json` | Déclare les deux environnements (= deux kernels) |
| `.github/build-environment.yml` | Dépendances pour lancer le build en CI |
| `.github/workflows/deploy.yml` | Workflow GitHub Actions |
| `content/` | Notebooks d'exemple inclus dans le site |

## Récupérer les fichiers statiques

Le workflow `Build JupyterLite` se déclenche :

- automatiquement à chaque push sur `main` ;
- manuellement via **Actions → Build JupyterLite → Run workflow** (vous pouvez
  choisir la branche source).

Il produit le site dans `dist/` puis :

1. **Pousse le contenu sur la branche `gh-pages`** (réécrite à chaque build).
   C'est cette branche que vous importez dans le GitLab interne.
2. **Publie un artefact `jupyterlite-static`** (zip téléchargeable depuis la
   page du run) — pratique pour récupérer les fichiers en une fois.

## Héberger sur GitHub Pages (optionnel)

La même branche `gh-pages` peut servir de source à GitHub Pages, sans rien
changer au workflow :

1. **Settings → Pages**
2. **Source** : *Deploy from a branch*
3. **Branch** : `gh-pages` / `(root)` → **Save**

Le site sera disponible sur `https://<utilisateur>.github.io/<repo>/`. Comme il
est servi sous un sous-chemin, le build inclut déjà tout en relatif — si jamais
un asset ne se charge pas, reconstruisez avec
`--base-url /<repo>/` (voir plus bas).

## Importer dans le GitLab interne

1. Récupérez les fichiers (clone de la branche `gh-pages` ou téléchargement de
   l'artefact zip).
2. Servez le contenu de façon statique (GitLab Pages, Nginx, etc.).
3. Ouvrez `index.html` (ou `lab/index.html` pour aller directement dans Lab).

> ℹ️ Si le site est servi sous un sous-chemin (ex.
> `https://gitlab.interne/groupe/projet/`), les chemins relatifs de JupyterLite
> fonctionnent généralement tels quels. En cas de souci, reconstruisez avec
> `jupyter lite build --base-url /groupe/projet/ …`.

## Ajouter des paquets

- **Python** : ajoutez le nom du paquet dans `environment-python.yml` (doit
  exister sur [emscripten-forge](https://emscripten-forge.org/)).
- **R** : ajoutez `r-<paquet>` dans `environment-r.yml`.

## Build local (optionnel)

```bash
micromamba create -n build-env -f .github/build-environment.yml
micromamba activate build-env
cp README.md content/
jupyter lite build --contents content --output-dir dist
```

> ⚠️ Le build local nécessite un accès réseau à `prefix.dev` /
> emscripten-forge (uniquement pour la construction, pas pour l'exécution).
