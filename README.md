# Révision Orale

Outil personnel de révision pour l'oral du bac de français.

**Site live :** https://teliamoreau.com

## Comment ça marche

1. Tu colles ton passage avec des `[crochets]` autour des phrases que tu veux analyser.
2. Tu peux ajouter une correction modèle après une barre `|` : `[meilleur des temps | antithèse, hyperbole]`.
3. Le site te guide en 3 étapes : Cadre (problématique, mouvements, intro) → Phrases (analyse phrase par phrase) → Synthèse (mini-conclusions, transitions, conclusion, ouverture).
4. Mode Simulation pour t'entraîner à parler en temps limité.

## Structure du projet

```
oraldefrancais/
├── index.html      Le site complet (HTML + CSS + JS dans un seul fichier)
├── favicon.svg     L'icône du site
└── README.md       Ce fichier
```

## Stack

HTML + CSS + JavaScript pur. Pas de framework, pas de build, pas de dépendances.
Pour modifier le site : ouvrir `index.html`, modifier, sauver, push sur GitHub.
Cloudflare Pages redéploie automatiquement.

## Données

Tous les textes que tu écris sont stockés dans `localStorage` du navigateur.
Cela veut dire :
- Tes textes restent sur l'appareil où tu les as créés.
- Si tu changes de téléphone ou vides le cache, ils sont perdus.
- **Pour les sauvegarder** : utilise le bouton "Exporter" (à venir, P1).

## Déploiement

Voir `DEPLOYMENT.md` pour la procédure complète.

## Roadmap

- **P0** ✓ Site déployé sur teliamoreau.com
- **P1** Boutons Correct/À revoir, mode Révision ciblée, suppression du champ "Analyse" dans la synthèse, 3ème mouvement optionnel, bouton Export/Import
- **P2** Refonte du design

---

Maintenu par Télia Moreau.
