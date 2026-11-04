---
name: fiches-revision
description: Transforme un cours en fiche de révision et en flashcards interactives dans une seule page web (onglets Carte mentale, Fiche et Flashcards). Utiliser dès que l'utilisateur veut réviser, apprendre ou mémoriser un cours, quelle que soit la matière, ou mentionne fiche de révision, flashcards, résumé de cours, carte mentale NotebookLM, examen, partiel, contrôle ou "aide-moi à retenir", même sans demander explicitement une fiche. La source peut être une capture de carte mentale, un PDF, une photo de cahier ou du texte collé.
---

# Fiches de révision interactives

Objectif : à partir d'un cours, produire une page web unique, utilisable sur téléphone, avec une **carte mentale** à branches repliables (comme dans NotebookLM), une **fiche** structurée et des **flashcards** retournables (« Je sais » / « À revoir »). L'utilisateur est étudiant et révise surtout sur mobile, donc l'interactif passe avant le texte long.

## Étapes

1. **Lire la source en entier.** Image ou capture : la regarder avec `view`. PDF : extraire le texte. Texte collé : l'utiliser tel quel. Si plusieurs sources, les fusionner dans un seul cours.
2. **Rester fidèle à la source.** Reprendre les notions, les formules et les chiffres du cours, sans en inventer. Si une partie manque (nœud replié d'une carte mentale, passage illisible), le dire à l'utilisateur à la fin au lieu de deviner. Un ajout venant de tes connaissances est permis seulement s'il aide à comprendre : le mettre dans un bloc `note` qui commence par « Complément : » pour qu'on voie qu'il ne vient pas du cours.
3. **Construire la carte mentale** (`map`) : un arbre avec le titre du cours au centre et trois niveaux de branches maximum, des libellés de 2 à 8 mots. Si la source est elle-même une carte mentale, recopier sa structure à l’identique (mêmes branches, mêmes noms). Sinon, la déduire du plan du cours. Les nœuds se replient au toucher grâce à la flèche.
4. **Construire la fiche** en suivant le plan du cours lui-même :
   - `summary` : 4 à 6 points qui résument l'essentiel.
   - `retenir` : 3 à 6 points à mémoriser absolument (formules, définitions, règles). Seulement ce qui vient du cours.
   - `conseils` : 3 à 5 conseils pratiques pour cette matière : pièges classiques, façon de s'entraîner, réflexes d'examen. Ce sont des conseils de méthode, pas de nouveaux faits sur le cours.
   - une section par grande partie, avec les types de blocs adaptés à la matière :
     - maths, recherche opérationnelle, physique : `def`, `formula`, `steps`, `example` (exemple résolu avec les données du cours)
     - programmation, web, bases de données : `def`, `list`, `example` avec du code entre accents graves, `tip` pour les erreurs classiques
     - réseaux, systèmes, cybersécurité : `list` pour les protocoles et commandes, `steps` pour les procédures, `tip` pour les pièges
     - théorie (gestion, droit, culture générale) : `def`, comparaisons en `list`, `tip`
5. **Construire les flashcards** : 12 à 30 selon la taille du cours, une idée par carte, réponse de 3 lignes maximum. Varier les types : définition, formule, étapes, « quelle différence entre A et B », « pourquoi », application sur un exemple du cours. `tag` = le titre de la section d'origine (sert de filtre).
6. **Générer la page** : copier `assets/template.html` vers `/mnt/user-data/outputs/fiche-<sujet>.html`, puis remplacer uniquement l'objet `COURSE` entre les repères `// <COURSE>` et `// </COURSE>`. Ne pas toucher au reste du template sans raison, il est déjà adapté au mobile, au mode sombre et à l'accessibilité.
7. **Livrer** : si l'outil Artifact est disponible, publier la page avec ; sinon la présenter avec `present_files`. Si l'utilisateur veut télécharger la page (pour réviser sans connexion ou la garder), la lui envoyer en fichier `.html` avec `present_files` : elle est autonome et s'ouvre dans n'importe quel navigateur sans internet. Proposer ce téléchargement en une phrase à la fin de la réponse. Ajouter 2 ou 3 lignes : ce que contient la fiche, et ce qui manquait dans la source. Proposer ensuite la suite (autre cours, ajout d'un quiz).

## Format de l'objet COURSE

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

Dans les textes : `**gras**` met en surbrillance, `` `code` `` met en police monospace, `\n` fait un retour à la ligne. Écrire les formules en texte lisible (x1, ≤, ≥, Σ, √) plutôt qu'en LaTeX, qui n'est pas affiché.

## Style

- Écrire en français, sauf si le cours est dans une autre langue. Utiliser l'apostrophe typographique ’ et les guillemets « » dans les textes.
- Phrases courtes, vocabulaire du cours. Une fiche se relit en cinq minutes.
- Ne jamais remplir une section pour faire joli : une partie courte reste courte.
