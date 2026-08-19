# Composer de Constitution — fond structuré

Ce dossier porte le **modèle de données** qui alimente l'application interactive
de composition de Constitution (vision Notion :
[App de configuration à la carte](https://app.notion.com/p/36e9776be4618197a48df9b1bb302911)).

- [`SCHEMA.md`](SCHEMA.md) — format machine (blocs de socle, modules, insertions, remplacements obligatoires, dépendances).
- [`constitution.fr.json`](constitution.fr.json) et [`constitution.en.json`](constitution.en.json) — Constitution structurée bilingue.
- [`glossaire.fr.json`](glossaire.fr.json) et [`glossaire.en.json`](glossaire.en.json) — glossaire bilingue.

Le fond est la **source de vérité** ; l'app (`dev/constitution-composer`) consomme
ce JSON, elle ne le réécrit pas. Le texte canonique humain reste les `.md` de
`../v6-alpha/`.

## Méthode : tranche verticale end-to-end

On valide l'accouplement fond ↔ app sur un petit périmètre réel, jouable, puis on
étend module par module. Jalons, chacun livré de bout en bout :

1. **J1 — Lecture + colorisation.** Rendu du socle, couleur par niveau (cœur / intégral / périphérie).
2. **J2 — Composition.** Boutons « + » / cases au fil du texte, activation de module → insertion animée en direct.
3. **J3 — Cohérence.** Insertions obligatoires de remplacement (`fallback`) + état *warning* visuel ; dépendances `requires`/`conflicts`.
4. **J4 — Navigation.** Sommaire latéral, ancrage vertical, compteur de modules actifs.
5. **J5 — Export.** Génération du Markdown composé → PDF (attribution + mention non officielle au pied).
6. **J6 — Freemium.** Seuil gratuit (≤ N modules, pas d'export) ; au-delà, compte obligatoire (Supabase) = collecte de leads.

## Garanties de synchronisation

Le dépôt public publie ce dossier avec `v6-alpha/`. Le Composer le vendorise en
sous-module : `npm run fond:check` compare ces quatre fichiers octet pour octet
et génère les principes depuis les Markdown canoniques. Toute divergence fait
échouer la CI.
