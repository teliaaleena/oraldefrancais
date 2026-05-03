# DEPLOYMENT.md

Document de référence pour le déploiement et la maintenance de **revisionorale** sur `teliamoreau.com`.

> **Si tu lis ça parce que quelque chose est cassé** : va directement à [Dépannage](#dépannage) en bas du fichier.

---

## Vue d'ensemble

Le site est un site statique (HTML + CSS + JavaScript pur, pas de build) hébergé gratuitement sur Cloudflare Pages, avec déploiement automatique depuis GitHub.

```
        ┌─────────────────────┐
        │  Télia édite        │
        │  index.html sur     │
        │  GitHub (web UI)    │
        └──────────┬──────────┘
                   │ push to main
                   ▼
        ┌─────────────────────┐
        │  GitHub repo        │
        │  teliaaleena/       │
        │  oraldefrancais     │
        └──────────┬──────────┘
                   │ webhook
                   ▼
        ┌─────────────────────┐
        │  Cloudflare Pages   │
        │  (compte Vincent)   │
        │  oraldefrancais     │
        │  .pages.dev         │
        └──────────┬──────────┘
                   │ DNS CNAME
                   ▼
        ┌─────────────────────┐
        │  teliamoreau.com    │
        │  (live, ~30s après  │
        │   le push)          │
        └─────────────────────┘
```

---

## Comptes et propriété

| Élément | Propriétaire | Compte |
|---|---|---|
| Code source | Télia | github.com/teliaaleena |
| Repo | Télia | github.com/teliaaleena/oraldefrancais (public) |
| Cloudflare Pages projet | Vincent | dash.cloudflare.com |
| Domaine `teliamoreau.com` | Vincent | dash.cloudflare.com (Registrar + DNS) |

**Pourquoi ce split** : Télia peut éditer le contenu sans toucher au compte Cloudflare. Vincent gère DNS et facturation domaine sans toucher au compte GitHub. Si plus tard Télia veut tout récupérer, le repo lui appartient déjà ; il suffit de transférer le projet Pages et le domaine sur son propre compte Cloudflare.

---

## Configuration Cloudflare Pages

Ces réglages sont fixés. Si quelqu'un les modifie, le site peut casser.

| Paramètre | Valeur |
|---|---|
| Project name | `oraldefrancais` |
| Repository | `teliaaleena/oraldefrancais` |
| Production branch | `main` |
| Framework preset | `None` |
| Build command | *(vide)* |
| Build output directory | `/` |
| Root directory | `/` |

**Custom domains attachés** :
- `teliamoreau.com` (apex, primary)
- `www.teliamoreau.com` (redirect to apex) *— à confirmer si configuré*

**DNS** (côté Cloudflare DNS, zone `teliamoreau.com`) :
- Type `CNAME`, Name `@`, Target `oraldefrancais.pages.dev`, Proxied (orange cloud)

---

## Comment modifier le site

### Cas 1 — petite modif (texte, couleur, copy)

1. Va sur `github.com/teliaaleena/oraldefrancais`
2. Clique le fichier à modifier (ex. `index.html`)
3. Bouton **crayon ✏️** en haut à droite
4. Modifie dans l'éditeur
5. En bas, dans "Commit changes" : écris un message court (ex. "Corrige typo intro")
6. Bouton vert **Commit changes**
7. Attends ~30 secondes
8. Recharge `teliamoreau.com` (Cmd+Shift+R / Ctrl+Shift+R pour vider le cache navigateur)

### Cas 2 — modif locale (préférable pour gros changements)

Si tu as un éditeur de code et que tu sais utiliser git :

```bash
git clone https://github.com/teliaaleena/oraldefrancais.git
cd oraldefrancais
# édite avec ton éditeur préféré
# teste localement en ouvrant index.html dans un navigateur
git add .
git commit -m "Description du changement"
git push origin main
```

### Vérifier qu'un déploiement a réussi

1. dash.cloudflare.com → Workers & Pages → projet `oraldefrancais`
2. Onglet **Deployments**
3. Le dernier deploy doit être "Success" (vert). Si "Failed" (rouge), clique pour voir les logs.

---

## Sauvegardes

**Le code** : sauvegardé automatiquement par GitHub. Chaque commit est récupérable. Pas d'action requise.

**Les textes que Télia écrit** : stockés dans `localStorage` du navigateur, **pas synchronisés**, **pas sauvegardés sur un serveur**. Si Télia change de téléphone ou vide son cache, ses textes sont perdus.

**Mitigation** : un bouton "Exporter / Importer" sera ajouté en P1. En attendant, Télia peut sauvegarder manuellement :
- Sur Chrome/Edge desktop : F12 → onglet "Application" → "Local Storage" → `https://teliamoreau.com` → clic droit sur `revision_texts` → copier la valeur → coller dans un fichier `.txt`

---

## Dépannage

### Le site affiche une page blanche ou erreur 404

**Vérifications dans l'ordre :**

1. **Le repo a-t-il bien `index.html` à la racine ?**
   - Va sur `github.com/teliaaleena/oraldefrancais`
   - Le fichier `index.html` doit être visible à la racine (pas dans un sous-dossier)
   - Si renommé ou déplacé, c'est la cause

2. **Le dernier déploiement Cloudflare a-t-il réussi ?**
   - dash.cloudflare.com → Workers & Pages → `oraldefrancais` → Deployments
   - Dernier deploy doit être vert ("Success")
   - Si rouge ("Failed"), clique pour voir les logs et corriger

3. **Le DNS pointe-t-il toujours bien ?**
   - dash.cloudflare.com → Websites → `teliamoreau.com` → DNS → Records
   - Doit avoir : `CNAME @ → oraldefrancais.pages.dev` (proxied)
   - Si manquant ou différent, le recréer

4. **Cache navigateur** : essayer en navigation privée. Si ça marche en privé mais pas en normal, c'est juste du cache local — pas un vrai problème.

### Le site marche sur `oraldefrancais.pages.dev` mais pas sur `teliamoreau.com`

Problème DNS, pas problème code.

1. dash.cloudflare.com → projet Pages `oraldefrancais` → onglet **Custom domains**
2. Vérifier que `teliamoreau.com` est listé avec statut "Active" (point vert)
3. Si "Verifying" ou "Error", cliquer "Complete DNS setup" et suivre les instructions
4. Si manquant, cliquer "Set up a custom domain" et entrer `teliamoreau.com`

### Une modif récente a cassé le site

**Rollback rapide** :

1. dash.cloudflare.com → projet `oraldefrancais` → Deployments
2. Trouver le dernier déploiement qui marchait (avant le bug)
3. Clic sur les "..." → **Rollback to this deployment**
4. Ça remet la version précédente en live en ~30 secondes

Le repo GitHub n'est pas modifié — il faut quand même corriger le code source ensuite. Mais au moins le site live est sauf le temps de réparer.

### Comment voir le code source d'une ancienne version

GitHub garde tout :
1. `github.com/teliaaleena/oraldefrancais` → bouton **Commits** (au-dessus de la liste de fichiers)
2. Cliquer sur un commit pour voir l'état du code à ce moment-là
3. Bouton "Browse files" pour explorer tout le repo à ce moment-là

### Le domaine `teliamoreau.com` ne marche plus du tout (même `oraldefrancais.pages.dev` non plus)

Ça veut dire que le compte Cloudflare a un problème (suspension, paiement, etc.).

1. Se connecter à dash.cloudflare.com avec les identifiants Vincent
2. Vérifier les notifications en haut de page
3. Vérifier l'onglet Billing pour s'assurer qu'aucun paiement n'a échoué (Cloudflare Pages est gratuit, mais le domaine a un coût annuel)

---

## Coûts

- **GitHub** : gratuit (repo public)
- **Cloudflare Pages** : gratuit (limite de 500 builds/mois et 100k requêtes/jour, on est très loin du seuil)
- **Domaine `teliamoreau.com`** : ~10 USD/an, facturé sur le compte Cloudflare de Vincent

---

## Stack technique

Pour mémoire, en cas de besoin de modification structurelle :

- **Pas de framework** (pas de React, Vue, Svelte, Next.js)
- **Pas de build step** (pas de webpack, vite, parcel)
- **Pas de package manager** (pas de npm, pas de `node_modules`)
- **Pas de dépendances externes au runtime** sauf :
  - Google Fonts (Sora + JetBrains Mono) chargées en CDN
  - Police de fallback : sans-serif système si Google Fonts down

Tout le code (HTML + CSS + JS) tient dans `index.html`. Si tu veux modifier le JavaScript, il est en bas du fichier dans une balise `<script>`. Le CSS est en haut dans `<style>`. Tout commence par `<!DOCTYPE html>`.

---

## Historique des décisions structurelles

- **3 mai 2026** : initial deploy. Migration depuis Replit (`revision-game--teliamoreau.replit.app`) vers Cloudflare Pages. Repo propre créé from scratch (l'ancien repo `oraldefrancaisrevision` contenait des artefacts Replit qui n'avaient rien à faire là).
- **Décision** : pas de login utilisateur. Données en `localStorage` uniquement, avec bouton Export/Import à venir en P1. Justification : login = serveur = point de panne supplémentaire = barrière à l'usage. Le coût en simplicité ne valait pas le bénéfice de sync auto pour un usage perso.
- **Décision** : single-file architecture (HTML+CSS+JS dans `index.html`). Justification : Télia n'est pas dev, doit pouvoir tout modifier en un seul endroit, dans GitHub web UI, sans changer d'onglet.

---

## Contacts

- **Code et contenu** : Télia Moreau
- **Infra (Cloudflare, domaine)** : Vincent Moreau
