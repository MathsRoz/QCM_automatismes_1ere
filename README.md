# QCM Mathématiques — Première

Générateur de QCM interactif pour les **automatismes de Première** (BO 2025, session 2027), conçu pour les professeurs et les élèves.

[https://mathsroz.github.io/QCM_automatismes_1ere/](https://mathsroz.github.io/QCM_automatismes_1ere/)

---

## Fonctionnalités

- **Génération aléatoire** de QCM à partir d'une banque de questions paramétriques
- **Seed déterministe** — chaque QCM est reproductible via un lien partageable
- **Mode présentation** plein écran pour le vidéoprojecteur (navigation clavier)
- **Export LaTeX** — document classique (article) ou diaporama (Beamer)
- **Compilation PDF** via [ytotech](https://latex.ytotech.com) ou Overleaf en fallback
- **Banque** (`editeur-banque.html`) pour consulter, tester et compiler les questions
- **Page question individuelle** (`question.html?id=…`) avec variables éditables et tirages alternatifs
- **Thème clair / sombre** persisté via `localStorage`

---

## Structure des fichiers

```
├── index.html              Application principale (génération de QCM)
├── editeur-banque.html     Éditeur et explorateur de la banque de questions
├── question.html           Affichage d'une question individuelle (?id=calc_001)
├── Mentions-Legales.html   Page mentions légales
├── style.css               Feuille de style commune (thème clair/sombre)
├── presentation.css        Feuille de style du mode présentation (diaporama)
├── compiler.js             Fonctions de compilation LaTeX partagées
├── presentation.js         Fonctions du mode présentation partagées
└── questions/
    ├── utils.js            Utilitaires (frac, ri, pickVar, dedupeAnswers, makeTable…)
    ├── svg-renderer.js     Moteur de rendu SVG + TikZ (objet Fig)
    ├── calcul.js           Thème : Calcul numérique et algébrique
    ├── proportions.js      Thème : Proportions et pourcentages
    ├── evolutions.js       Thème : Évolutions et variations
    ├── fonctions.js        Thème : Fonctions et représentations
    ├── stats.js            Thème : Statistiques
    ├── proba.js            Thème : Probabilités
    └── autre.js            Questions en cours / non répertoriées
```

---

## Banque de questions

La banque contient actuellement environ **90 questions actives** réparties en 6 thèmes.

### Calcul numérique et algébrique (`calcul.js`)

| Groupe | IDs |
|---|---|
| Opérations et comparaisons de fractions | C1-A … C1-D2 |
| Opérations sur les puissances | C2-A, C2-B |
| Écriture décimale, fractionnaire et pourcentage | C3-A, C3-B |
| Développer, factoriser | C4-A … C4-D |
| Résoudre une équation ou inéquation | C5-A … C5-E |
| Isoler une variable dans une égalité | C6-x |
| Simplifier une expression littérale | C7-x |
| Déterminer le signe d'une expression | C8-x |
| Passer du français aux mathématiques | C9-x |
| Effectuer des conversions d'unités | C10-x |
| Estimer un ordre de grandeur | C11-x |

### Proportions et pourcentages (`proportions.js`)

| Groupe | IDs |
|---|---|
| Calculer, appliquer, exprimer une proportion | R1-A … R1-F |
| Calculer une proportion de proportion | R2-A, R2-B |

### Évolutions et variations (`evolutions.js`)

| Groupe | IDs |
|---|---|
| Passer d'une formulation additive à multiplicative | E1-A, E1-B, E1-C |
| Appliquer un taux d'évolution | E2-A … E2-C |
| Calculer un taux d'évolution | E3-A |
| Calculer le taux d'évolutions successives | E4-A … E4-C |
| Calculer un taux d'évolution réciproque | E5-A |

### Fonctions et représentations (`fonctions.js`)

| Groupe | IDs |
|---|---|
| Exploiter une équation de courbe | F1-A |
| Représentation de droites et fonctions affines | F2-A … |
| Calculer une image | F3-x |
| Calculer le coefficient directeur | F4-x |
| Déterminer graphiquement images et antécédents | F5-x |
| Résoudre graphiquement une équation / inéquation | F6-x |
| Tableau de signes et de variations | F7-x |

### Statistiques (`stats.js`)

| Groupe | IDs |
|---|---|
| Calculer et interpréter des indicateurs (moyenne, médiane, quartiles, étendue) | S1_A … S1_E |
| Lire et commenter des graphiques usuels | S2_A |

### Probabilités (`proba.js`)

| Groupe | IDs |
|---|---|
| Probabilité de l'événement contraire | P1-A |
| Calcul dans une situation d'équiprobabilité | P2-A … P2-E |
| Probabilités conditionnelles | P3-A … P3-C |
| Calcul dans un arbre | P4-A … P4-D |
| Calcul dans un tableau | P5-A |

---

## Format d'une question

```js
{
  id: 'calc_001',                          // identifiant unique
  theme: 'calcul',                         // thème (calcul, proportions, evolutions, fonctions, stats, proba)
  groupe: 'fractions',                     // groupe thématique (évite les doublons lors de la génération)
  desc: 'Addition de fractions',           // courte description (affichée dans l'éditeur)
  niveau: ['techno', 'specifique', 'specialite'],
  cols: 4,                                 // nombre de colonnes pour l'affichage des réponses (1–4)

  variables: {
    a: { min: 1, max: 9 },                 // entier aléatoire dans [min, max]
    b: { min: 2, max: 5, step: 2 },        // avec pas
    c: { values: [10, 20, 50, 100] },      // liste de valeurs possibles
  },

  enonce:       (v) => `Calculer $\\dfrac{${v.a}}{${v.b}}$`,
  bonneReponse: (v) => `$${frac(v.a, v.b)}$`,
  distracteurs: (v) => [`$${v.a}$`, `$${v.b}$`, `$${v.a + v.b}$`],
}
```

### Insérer une figure SVG + TikZ

```js
enonce: (v) => {
  if (v._deduping) return ''; // évite de générer la figure lors de la déduplication
  const svg  = Fig.svg(-3, 3, -2, 2).axes().curve('x^2-1').end();
  const tikz = Fig.latex(-3, 3, -2, 2).axes().curve('x^2-1').end();
  return 'Soit la courbe ci-dessous :'
    + '%%SVG' + svg + '%%ENDSVG%%%%TIKZ' + tikz + '%%ENDTIKZ%%'
    + 'Quel est le minimum ?';
}
```

Les marqueurs `%%SVG…%%ENDSVG%%%%TIKZ…%%ENDTIKZ%%` permettent un double rendu : SVG pour le navigateur, TikZ pour l'export LaTeX.

---

## API Fig (`svg-renderer.js`)

Objet fluide pour générer des figures mathématiques en SVG (navigateur) et TikZ (LaTeX).

```js
// Initialisation
Fig.svg(xmin, xmax, ymin, ymax)            // contexte SVG navigateur
Fig.latex(xmin, xmax, ymin, ymax, scale)   // contexte TikZ LaTeX

// Méthodes chaînables
.axes()                                    // repère avec flèches
.grid(stepx, stepy)                        // quadrillage
.gradX(step) / .gradY(step)               // graduations sur les axes
.curve('x^2 - 2*x + 1')                   // courbe d'une expression
.affine(a, b, xmin, xmax, color, label)   // droite y = ax + b
.line(x1, y1, x2, y2, color, style)       // segment (style: 'dashed', 'dotted'…)
.point(x, y, label, color, anchor)        // point avec label
.text(x, y, 'texte', color, anchor)       // texte (supporte $x_1$, ~x pour barre)
.frac(x, y, num, den)                     // fraction centrée
.label(expr, 'C_f', color)                // étiquette sur une courbe
.dashes(x0, y0, labelX, labelY)           // tirets de lecture graphique
.arbre()                                   // arbre de probabilités (2 niveaux, A/Ā × B/B̄)
.camembert([{val, color}])                 // diagramme circulaire
.tableauS(t)                               // tableau de signes
.tableauV(t)                               // tableau de variations
.clip() / .endClip()                       // zone de découpe
.end()                                     // retourne la chaîne SVG ou TikZ
```

Les marges internes sont configurables via `Fig._PAD = { l, r, t, b }`.

**Fonction utilitaire :** `fimage(a, expr)` évalue `expr` en `x = a` en utilisant le parseur de Fig (ex: `fimage(3, 'x^2-1')` → `8`).

---

## Utilitaires (`utils.js`)

| Fonction | Description |
|---|---|
| `ri(min, max)` | Entier aléatoire dans [min, max] |
| `rf(min, max, dec)` | Flottant aléatoire arrondi à `dec` décimales |
| `riStep(min, max, step, values)` | Tirage avec pas ou liste de valeurs |
| `pickVar(cfg)` | Tire une valeur selon la config d'une variable |
| `nonZero(min, max)` | Entier aléatoire non nul |
| `pgcd(a, b)` | Plus grand commun diviseur |
| `frac(num, den)` | Fraction LaTeX simplifiée (`\dfrac{…}{…}`) |
| `simplExpr(s)` | Simplifie les expressions algébriques (1x → x, +0 → …) |
| `dedupeAnswers(template, vars)` | Régénère les variables jusqu'à éliminer les doublons entre bonne réponse et distracteurs |
| `fimage(a, expr)` | Image d'un nombre par une expression |
| `makeTable(rows, opts)` | Génère un tableau HTML + LaTeX à partir d'un tableau de lignes |

---

## Seed et reproductibilité

Chaque QCM généré reçoit une **seed** affichée comme badge `🎲 XXXXXXXXX`. Cliquer dessus copie un lien du type :

```
index.html?seed=482759301&level=specialite&inclusive=1&n=12&themes=calcul,proba
```

Ouvrir ce lien régénère exactement le même QCM (mêmes questions, mêmes variables, même ordre). Le générateur utilisé est **Mulberry32**, patché sur `Math.random` uniquement pendant la génération, puis restauré.

---

## Export LaTeX (`compiler.js`)

### Document classique (article)
- Questions numérotées avec réponses en colonnes (`multicols`)
- Figures en `minipage` côte à côte avec le texte
- Option : inclure le corrigé en fin de document

### Diaporama (Beamer)
- Formats : question seule / question + correction intercalée / toutes les questions puis toutes les corrections
- Choix du thème Beamer (Singapore, Madrid, Warsaw…) et de la palette de couleurs
- Figures en `columns` côte à côte

### Compilation
1. **Automatique** via [latex.ytotech.com](https://latex.ytotech.com) → téléchargement direct du PDF
2. **Fallback** → ouverture sur [Overleaf](https://overleaf.com) avec le code pré-rempli

---

## Mode présentation (`presentation.js` + `presentation.css`)

Activé depuis l'interface principale, le mode présentation s'ouvre en plein écran. Navigation :

| Touche | Action |
|---|---|
| `→` ou `Espace` | Question suivante |
| `←` | Question précédente |
| `C` | Afficher / masquer le corrigé |
| `Échap` | Fermer |

La taille de police s'adapte automatiquement à la hauteur de l'écran (`adaptPresFont`).

---

## Ajouter des questions

1. Ouvrir le fichier du thème correspondant (ex: `questions/stats.js`)
2. Ajouter un objet question dans le tableau `QUESTIONS_STATS`
3. Tester via `question.html?id=votre_id` ou dans l'éditeur de banque (`editeur-banque.html`)

Les variables sont automatiquement dé-dupliquées par `dedupeAnswers` : si la bonne réponse et un distracteur sont identiques, de nouvelles variables sont tirées (jusqu'à 10 tentatives). Le flag `v._deduping` permet aux questions avec figures lourdes de sauter la génération SVG pendant cette phase.

---

## Dépendances

| Bibliothèque | Usage | Version |
|---|---|---|
| [KaTeX](https://katex.org) | Rendu des formules mathématiques | 0.16.9 |
| [DM Sans / DM Mono](https://fonts.google.com) | Typographie | — |

Tout le reste est vanilla JS / HTML / CSS, sans framework ni bundler.

---

## Licence

[AGPL v3](https://www.gnu.org/licenses/agpl-3.0.html) — voir `Mentions-Legales.html` pour les détails.
