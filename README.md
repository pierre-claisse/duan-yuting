# duan

Site web de 段 (cours particuliers) : **vitrine** et **blog**.
Application React déployée sur GitHub Pages, qui persiste ses données
gratuitement dans un dépôt GitHub via un *personal access token* chiffré —
aucun serveur. Modèle technique inspiré de `hanzi-ruby-lens` (sans la
réactivité temps réel webhooks/Cloudflare).

> **Site dépublié.** https://duan.life ne sert plus l'application, mais une page
> blanche (voir [Déploiement](#déploiement)). Le code reste intact ici.

## Langues

Interface **bilingue permanente** : chaque libellé s'affiche en
**中文 / English** simultanément — il n'y a pas de sélecteur de langue.
Les chaînes sont dans [`src/i18n/translations.ts`](src/i18n/translations.ts) ;
les dates sont formatées via `Intl` selon la locale.

## Sections

- **Accueil** (`/`) — vitrine (placeholder neutre pour l'instant).
- **Blog** (`/blog`, `/blog/:slug`) — public en lecture ; la prof, une fois
  connectée, rédige / édite / supprime les articles. Contenu en **Markdown**.

## Architecture des données

Deux dépôts GitHub :

| Dépôt | Visibilité | Contenu | Accès |
|---|---|---|---|
| `duan` | public | Code de l'app | — |
| `duan-blog` | **public** | `articles/<slug>.json` + `articles/index.json` | lecture anonyme (raw), écriture via PAT |

- Lecture publique du blog : `raw.githubusercontent.com` (anonyme, sans quota ;
  cache CDN ~5 min — un article tout juste publié peut n'apparaître qu'après ce
  court délai).
- Écriture et lecture authentifiée : **GitHub Contents API** avec le PAT (lecture
  du SHA puis `PUT`). Une seule éditrice ⇒ pas d'IndexedDB ni d'orchestrateur de
  conflits.
- **Authentification** : le PAT est chiffré (Argon2id + AES-GCM) dans
  `public/secrets.json`, généré au build par `scripts/build-secrets.mjs`. La prof
  saisit un mot de passe pour le déchiffrer ; le PAT reste en mémoire (jamais en
  localStorage, jamais en clair sur disque).

## Développement

```sh
npm install
npm run dev        # http://localhost:5173 — Accueil + Blog fonctionnent sans secrets
npm run typecheck  # tsc --noEmit
npm run build      # bundle de production dans dist/
```

Pour tester la **connexion prof** en local, générer le bundle de secrets
(gitignoré) :

```sh
SYNC_PAT='github_pat_…' SYNC_PASSWORD='…' node scripts/build-secrets.mjs
```

## Déploiement

**État actuel : dépublié.** Push sur `main` → GitHub Actions
([`.github/workflows/blank.yml`](.github/workflows/blank.yml)) publie tel quel le
dossier [`offline/`](offline/) : une page vide en `noindex`, un `404.html`
identique (pour que les anciennes URL type `/blog` restent blanches elles aussi),
un `robots.txt` qui interdit tout, et un `CNAME`. Ni `npm`, ni build, ni
`build-secrets.mjs` — le bundle chiffré `public/secrets.json` n'est donc plus
publié.

Le `CNAME` est conservé volontairement : il garde `duan.life` rattaché à ce dépôt.
Sans lui, les enregistrements DNS pointeraient vers GitHub Pages sans propriétaire,
et n'importe qui pourrait revendiquer le domaine.

Le pipeline de l'application est intact mais inerte, dans
[`.github/workflows/deploy.yml.disabled`](.github/workflows/deploy.yml.disabled)
(Actions n'exécute que les fichiers `.yml` / `.yaml`) :
`npm ci` → `node scripts/build-secrets.mjs` → `npm run build` (base `/`,
domaine personnalisé via [`public/CNAME`](public/CNAME)) → déploiement Pages.

**Pour republier l'application :** renommer `deploy.yml.disabled` en `deploy.yml`,
supprimer `blank.yml` et `offline/`, vérifier que les secrets CI `SYNC_PAT` et
`SYNC_PASSWORD` sont toujours valides sur le dépôt `duan`, puis push sur `main`.
Les coordonnées du dépôt de données sont publiques et vivent dans
[`src/config.ts`](src/config.ts).
