# Le Fil de Bretagne — Notes de déploiement

## Stack
- `index.html` fichier unique (pas de build)
- Tailwind CSS CDN + Google Fonts
- Photos Pexels (IDs réels vérifiés)
- GitHub Pages via GitHub Actions

## Procédure déploiement GitHub Pages (à suivre dans l'ordre)

### Étape 1 — Activer Pages manuellement EN PREMIER
Avant tout push du workflow, aller dans :
**Settings → Pages → Build and deployment → Source → "GitHub Actions"**

> ⚠️ Sans cette étape préalable, `deploy-pages` échoue silencieusement (workflow vert, 404 sur le site).
> `configure-pages enablement: true` n'est pas fiable pour la première activation.

### Étape 2 — Workflow en 2 jobs séparés (obligatoire)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

> ⚠️ L'environnement `github-pages` doit être sur le job `deploy` SÉPARÉ du build.
> Un job unique fait échouer silencieusement le déploiement.

### Étape 3 — Permissions workflow
```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### Étape 4 — Repo public obligatoire
GitHub Pages gratuit = repo public uniquement.

### URL finale
```
https://{username-lowercase}.github.io/{RepoName}/
```

### Checklist complète
1. [ ] Repo public
2. [ ] Settings → Pages → Source → "GitHub Actions" (AVANT le premier push du workflow)
3. [ ] Workflow en 2 jobs (build + deploy)
4. [ ] `permissions: pages: write + id-token: write`
5. [ ] Push sur la branche configurée dans `on.push.branches`
