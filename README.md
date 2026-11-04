# fiches-revision

Un skill Claude qui transforme un cours en **page web de révision interactive** : une carte mentale, une fiche structurée et des flashcards, le tout dans un seul fichier HTML qui marche sur téléphone, y compris sans connexion.

## Ce que le skill produit

À partir d’un cours (capture de carte mentale, PDF, photo de cahier ou texte collé), Claude génère une page unique avec trois onglets :

| Onglet | Contenu |
|---|---|
| **Carte mentale** | Arbre à branches repliables (3 niveaux maximum), avec le titre du cours au centre. Si la source est déjà une carte mentale (NotebookLM par exemple), sa structure est recopiée à l’identique. |
| **Fiche** | Résumé (4 à 6 points), « À retenir » (3 à 6 points), conseils de méthode (3 à 5), puis une section repliable par grande partie du cours. |
| **Flashcards** | 12 à 30 cartes retournables avec « Je sais » / « À revoir », barre de progression, filtre par partie, mode « À revoir seulement » et mélange des cartes. |

La page s’adapte au mobile et au mode sombre. Elle respecte les bases d’accessibilité (rôles `tab`, `aria-expanded`, focus visible). La progression des flashcards est enregistrée dans le navigateur (`localStorage`).

## Déclenchement

Le skill s’active quand tu veux réviser, apprendre ou mémoriser un cours, ou que tu mentionnes une fiche de révision, des flashcards, un résumé de cours, une carte mentale, un examen, un partiel, un contrôle ou « aide-moi à retenir ». Aucune matière n’est imposée.

## Installation

1. Télécharge [`fiches-revision.skill`](./fiches-revision.skill).
2. Dans Claude, ouvre **Paramètres → Capacités → Skills** et importe le fichier.
3. Vérifie que le skill est activé.

Pour un usage avec Claude Code, copie le dossier `fiches-revision/` dans `~/.claude/skills/` (ou dans `.claude/skills/` à la racine d’un projet).

## Utilisation

Envoie ton cours à Claude, avec ou sans consigne :

- « Fais-moi une fiche de révision de ce cours » + un PDF ou une photo.
- « Aide-moi à retenir ce chapitre » + du texte collé.
- Une capture de carte mentale suffit : Claude la reprend telle quelle et en tire la fiche et les flashcards.

Claude livre la page (publiée en lien ou en fichier `.html`), résume ce qu’elle contient et signale ce qui manquait dans la source (nœud replié, passage illisible). Il ne devine pas les parties absentes.

## Règles de fidélité

- Les notions, formules et chiffres viennent du cours, rien n’est inventé.
- Un ajout issu des connaissances de Claude n’est permis que s’il aide à comprendre, et il apparaît dans un bloc `note` qui commence par « Complément : ».
- Les conseils de méthode portent sur la façon de réviser, pas sur de nouveaux faits.

## Structure du dépôt

```
.
├── README.md
├── LICENSE                      # licence MIT
├── fiches-revision.skill        # archive prête à importer
└── fiches-revision/
    ├── SKILL.md                 # instructions du skill
    └── assets/
        └── template.html        # page modèle (CSS, JS, onglets, flashcards)
```

## Fonctionnement interne

Claude copie `assets/template.html` vers `fiche-<sujet>.html`, puis remplace **uniquement** l’objet `COURSE` situé entre les repères `// <COURSE>` et `// </COURSE>`. Le reste du template (mise en page, mode sombre, logique des onglets et des cartes) n’est pas modifié.

### Format de l’objet `COURSE`

```js
const COURSE = {
  title: "Titre du cours",
  subtitle: "Matière, chapitre ou date d'examen",
  summary: ["Point clé avec **mot important** en surbrillance"],
  retenir: ["Ce qu'il faut absolument retenir"],
  conseils: ["Conseil pour réviser ou éviter un piège"],
  sections: [{ title: "Nom de la partie", blocks: [
    { type: "def", term: "Terme :", text: "définition" },
    { type: "formula", label: "Nom de la formule", text: "Z = c1x1 + … + cnxn" },
    { type: "steps", title: "Méthode", steps: ["étape 1", "étape 2"] },
    { type: "example", title: "Exemple 1", text: "énoncé et résolution" },
    { type: "tip", text: "piège à éviter" },
    { type: "list", items: ["a", "b"] },
    { type: "note", text: "Complément : ..." }
  ]}],
  map: { label: "Titre du cours", children: [{ label: "Partie", children: [{ label: "Notion" }] }] },
  cards: [{ tag: "Nom de la partie", q: "Question ?", a: "Réponse courte" }]
};
```

### Types de blocs

| Type | Usage | Champs |
|---|---|---|
| `def` | Définition | `term`, `text` |
| `formula` | Formule, affichée en police monospace | `label`, `text` |
| `steps` | Méthode ou procédure en étapes | `title`, `steps[]` |
| `example` | Exemple résolu avec les données du cours | `title`, `text` |
| `tip` | Piège à éviter (bordure rouge) | `text` |
| `list` | Liste à puces | `items[]` |
| `note` | Complément, en italique | `text` |

### Mise en forme du texte

- `**gras**` : surbrillance
- `` `code` `` : police monospace
- `\n` : retour à la ligne
- Les formules s’écrivent en texte lisible (x1, ≤, ≥, Σ, √), car le LaTeX n’est pas affiché.

### Blocs conseillés selon la matière

| Matière | Blocs |
|---|---|
| Maths, recherche opérationnelle, physique | `def`, `formula`, `steps`, `example` |
| Programmation, web, bases de données | `def`, `list`, `example` (code entre accents graves), `tip` |
| Réseaux, systèmes, cybersécurité | `list`, `steps`, `tip` |
| Théorie (gestion, droit, culture générale) | `def`, `list` (comparaisons), `tip` |

## Style rédactionnel

- Français, sauf si le cours est dans une autre langue.
- Apostrophe typographique ’ et guillemets « ».
- Phrases courtes, vocabulaire du cours, une fiche qui se relit en cinq minutes.
- Pas de remplissage : une partie courte reste courte.

## Personnalisation

- **Apparence** : modifie les variables CSS au début de `assets/template.html` (`--ac` pour la couleur d’accent, `--mk` pour le surlignage, etc.).
- **Nombre de cartes ou de points** : ajuste les fourchettes dans `SKILL.md`.
- **Nouveau type de bloc** : ajoute son rendu dans le JavaScript du template, puis décris-le dans `SKILL.md`.

Après toute modification, régénère l’archive :

```bash
zip -r fiches-revision.skill fiches-revision
```

## Licence

Projet publié sous licence [MIT](./LICENSE).
