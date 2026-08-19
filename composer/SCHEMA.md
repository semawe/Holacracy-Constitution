# Modèle de données — Composer de Constitution

Format machine du texte composable. **Source de vérité du fond** : ce dossier
vit dans le repo `holacracy-constitution` ; l'app (`constitution-composer`) le
*consomme*, elle ne le réinvente pas. Le texte canonique reste les `.md` de
`v6-alpha/` ; ce JSON en est l'encodage applicatif.

## Principe

Une Constitution composée = un **socle** toujours présent + un jeu de **modules**
activables. Activer un module insère son texte à un point d'ancrage précis du
socle. Chaque morceau de texte porte un **niveau** (`tier`) qui détermine sa
couleur à l'écran et son appartenance (socle / bloc retirable / extension / app).

## Modèle V1 — on part de la *Lite*, on retire

V1 est bâtie sur la **Lite** (`v6-alpha/fr/HC-v6-lite.md`) : un **socle
incompressible** (les `block`, `always: true`) plus des **blocs retirables**
**cochés par défaut**. Un bloc retirable est techniquement un `module` de tier
`retirable` portant `default: true` : présent au départ (la Lite complète),
décochable pour alléger. Décocher un bloc qui porte un `fallback` insère une
**règle par défaut** (issue de l'*Adoption Declaration*, art. 2/3/4/5) pour
garder le texte cohérent.

Au-delà de la Lite, deux familles de modules **additifs** (`default: false`,
off au départ) : les **extensions** constitutionnelles (`extension`, ex. Agents)
et les **apps** (`app`). Tout coché → on tend vers l'intégrale.

Les trois fallbacks réels de la V1 : `scribe` (le Leader de Cercle assume),
`reunions-tactiques` (habitudes de réunion actuelles), `decision-integrative`
(le Leader de Cercle édite seul la Gouvernance). Les autres retirables
disparaissent sans règle de remplacement.

## Niveaux (`tier`) — colorisation

| tier         | sens                                                      | toujours présent |
|--------------|-----------------------------------------------------------|------------------|
| `core`       | le cœur non négociable (socle micro)                      | oui              |
| `retirable`  | bloc de la Lite, présent par défaut, décochable           | non (décochable) |
| `extension`  | du texte constitutionnel au-delà de la Lite (ex. Agents)  | non (activable)  |
| `app`        | une fonction hors constitution de base (ex. revue appréciative, décision rémunérations) | non (activable)  |

Les modules `app` portent du contenu **brouillon** (exemples fournis par Aliocha,
à détailler) ; ils sont marqués « *Brouillon, à détailler.* » dans le texte.

Un quatrième état *visuel* existe sans être un `tier` : l'**insertion de
remplacement** (`fallback`), affichée en mode *warning* quand elle se déclenche.

## Objets

### `block` — unité de socle

```jsonc
{
  "id": "article-1",          // identifiant stable
  "type": "article",          // preamble | article | section
  "anchor": "article-1",      // point d'ancrage citable par les modules
  "tier": "core",
  "always": true,             // fait partie du socle, jamais retiré
  "heading": "Article 1 : rôles et cercles",
  "intent": "…",              // note d'intention (affichable/masquable)
  "text": "…"                 // corps normatif (Markdown léger)
}
```

### `module` — unité activable

```jsonc
{
  "id": "representant-cercle",
  "label": "Représentant de Cercle",
  "tier": "retirable",
  "default": true,             // coché au départ ? true = bloc retirable de la Lite ; false/absent = additif (extension/app)
  "description": "…",          // texte de la case à cocher / tooltip
  "requires": [],              // modules prérequis (active en cascade)
  "conflicts": [],             // modules mutuellement exclusifs
  "insertions": [              // ce qui s'insère quand le module est ACTIF
    { "anchor": "article-1", "position": "append", "text": "…" },
    // insertion conditionnelle : n'apparaît que si TOUS ces modules sont aussi actifs
    { "anchor": "article-1", "position": "append", "whenActive": ["representant-cercle"], "text": "…" }
  ],
  "fallback": {                // ce qui s'insère quand le module est INACTIF
    "anchor": "article-1",     // (null si rien) — garantit la cohérence du texte
    "text": "…"
  }
}
```

## Règles de composition

1. **Socle d'abord.** Tous les `block` (`always: true`) sont rendus dans l'ordre.
2. **Insertions.** Pour chaque module actif, ses `insertions` sont placées à leur
   `anchor` (`append` = après le bloc ancre ; `after:<id>` possible plus tard).
3. **Remplacement obligatoire.** Pour chaque module *inactif* qui porte un
   `fallback`, le texte de remplacement est inséré à son `anchor`. C'est ce qui
   rend toute combinaison de cases cohérente et complète. Exemple : module
   `gouvernance-formelle` inactif → insertion par défaut « les rôles du Cercle
   sont définis par le Leader de Cercle ».
4. **Dépendances.** Cocher un module coche ses `requires` ; décocher un module
   décoche ceux qui le `requires`. Deux modules en `conflicts` ne peuvent être
   actifs ensemble.

## Export

La version composée (socle + insertions résolues, dans l'ordre) est sérialisable
en Markdown puis en PDF. L'attribution CC BY-SA 4.0 et la mention « version non
officielle » sont ajoutées au pied de chaque export (cf. vision Notion).
