# Compte rendu — TP Vue : Quiz en images

**Étudiant :** DhiaEddine Rezgani — **Groupe :** L2DSI1  
**Thème choisi :** formes géométriques (triangle, carré, cercle, étoile, rectangle)

## 1. Interface, éléments HTML et directives Vue
- `<header>` : titre et score en haut à droite (`{{ score }} / {{ questions.length }}`).
- `<progress :value="reponsesDonnees" :max="questions.length">` : barre de progression.
- `<img :src="..." :alt="...">` : image de la question avec texte alternatif.
- `<button v-for="(c, i) in questionCourante.choix" :key="i">` : exactement trois boutons de choix.
- `@click` : réponse, question suivante, rejouer. `:disabled` : bloque les boutons après la réponse.
- `v-if / v-else` : alterne l'écran du quiz et l'écran de résultat. `v-if` affiche aussi le message de correction et le bouton « Question suivante » / « Voir le résultat ».

## 2. Tableau des questions
Le tableau `questions` (dans `<script setup>` de `App.vue`) contient 5 objets :
`{ texte, image, choix: [3 chaînes], bonneReponse: 0|1|2, descriptionImage }`.
`image` est le chemin local (`/images/triangle.svg`, fichier dans `public/images`) et `descriptionImage` alimente l'attribut `alt`. La bonne réponse n'est pas toujours au même emplacement (indices 0, 1, 2, 1, 0).

## 3. Événements, score et progression
- État réactif : `indexQuestion`, `score`, `reponseChoisie` (`null` tant qu'on n'a pas répondu), `termine`.
- Le clic appelle `repondre(i)` : `i` (fourni par `v-for`) est l'indice du bouton cliqué, comparé à `bonneReponse`.
- Garde : `if (reponseChoisie.value !== null) return` → un seul point par question, même en cas de clics multiples ; de plus les boutons sont `:disabled`.
- **Score** = nombre de bonnes réponses. **Progression** = nombre de réponses données (computed `reponsesDonnees`), qui augmente aussi pour une mauvaise réponse.
- Dernière question : `derniereQuestion` (computed) change le bouton en « Voir le résultat » ; `suivante()` met `termine` à `true` sans dépasser le tableau. `rejouer()` remet tout à zéro.
- Après la réponse : message « ✔ Bonne réponse » / « ✘ Mauvaise réponse… » et symboles ✔ ✘ sur les boutons (la couleur n'est pas le seul indicateur).

## 4. Tests réalisés
| Test | Résultat |
|---|---|
| Nouvelle partie | Question 1, 3 choix, score 0/5, progression 0/5 |
| Bonne réponse | Score +1, progression +1 |
| Mauvaise réponse | Score inchangé, progression +1 |
| Clics multiples | Aucun point supplémentaire, boutons désactivés |
| Question suivante | Image, texte et choix mis à jour, score conservé |
| Cinq bonnes réponses | Score 5/5, progression 5/5 |
| Cinq mauvaises réponses | Score 0/5, progression 5/5 |
| Mélange | Score = nombre de bonnes réponses (ex. 4/5) |
| Rejouer | Retour à la question 1, score et progression à 0 |
| Petit écran | Mise en page fluide, boutons accessibles |

**Difficultés rencontrées :** distinguer le score (bonnes réponses) de la progression (réponses données) ; empêcher le double comptage d'un point (garde dans `repondre()` et boutons désactivés) ; ne pas dépasser le tableau à la dernière question ; indiquer juste/faux autrement que par la couleur seule.

## Captures
Voir le dossier `captures/`.
