<script setup>
import { ref, computed } from 'vue'

// Les 5 questions : texte, image, 3 choix, indice (0, 1 ou 2) de la bonne réponse
const questions = [
  { texte: 'Combien de côtés possède cette forme ?', image: '/images/triangle.svg',
    choix: ['Trois', 'Quatre', 'Cinq'], bonneReponse: 0,
    descriptionImage: 'Un triangle orange' },
  { texte: 'Quelle est la couleur de cette forme ?', image: '/images/carre.svg',
    choix: ['Rouge', 'Bleu', 'Vert'], bonneReponse: 1,
    descriptionImage: 'Un carré bleu' },
  { texte: 'Quelle est cette forme ?', image: '/images/cercle.svg',
    choix: ['Un ovale', 'Un carré', 'Un cercle'], bonneReponse: 2,
    descriptionImage: 'Un cercle vert' },
  { texte: 'Combien de branches possède cette étoile ?', image: '/images/etoile.svg',
    choix: ['Quatre', 'Cinq', 'Six'], bonneReponse: 1,
    descriptionImage: 'Une étoile jaune à cinq branches' },
  { texte: 'Quelle est cette forme ?', image: '/images/rectangle.svg',
    choix: ['Un rectangle', 'Un triangle', 'Un cercle'], bonneReponse: 0,
    descriptionImage: 'Un rectangle violet' },
]

// État réactif
const indexQuestion = ref(0)     // question affichée (0 à 4)
const score = ref(0)             // nombre de bonnes réponses
const reponseChoisie = ref(null) // indice du bouton cliqué (null = pas encore répondu)
const termine = ref(false)       // vrai quand l'écran de résultat est affiché

// Valeurs calculées
const questionCourante = computed(() => questions[indexQuestion.value])
const aRepondu = computed(() => reponseChoisie.value !== null)
const derniereQuestion = computed(() => indexQuestion.value === questions.length - 1)
const estCorrecte = computed(() => reponseChoisie.value === questionCourante.value.bonneReponse)
// Progression = nombre de réponses DONNÉES (pas de bonnes réponses)
const reponsesDonnees = computed(() => {
  if (termine.value) return questions.length
  return indexQuestion.value + (aRepondu.value ? 1 : 0)
})

function repondre(indice) {
  if (aRepondu.value) return // garde : une seule réponse comptée par question
  reponseChoisie.value = indice
  if (indice === questionCourante.value.bonneReponse) score.value++
}

function suivante() {
  if (!aRepondu.value) return // impossible d'avancer sans avoir répondu
  if (derniereQuestion.value) {
    termine.value = true
  } else {
    indexQuestion.value++
    reponseChoisie.value = null
  }
}

function rejouer() {
  indexQuestion.value = 0
  score.value = 0
  reponseChoisie.value = null
  termine.value = false
}

// Classe et symbole d'un bouton après la réponse (la couleur n'est pas le seul indicateur)
function classeChoix(i) {
  if (!aRepondu.value) return ''
  if (i === questionCourante.value.bonneReponse) return 'juste'
  if (i === reponseChoisie.value) return 'faux'
  return 'neutre'
}
function symboleChoix(i) {
  if (!aRepondu.value) return ''
  if (i === questionCourante.value.bonneReponse) return '✔ '
  if (i === reponseChoisie.value) return '✘ '
  return ''
}
</script>

<template>
  <main class="quiz">
    <header class="haut">
      <h1>Quiz des formes</h1>
      <p class="score" aria-live="polite">Score : {{ score }} / {{ questions.length }}</p>
    </header>

    <label class="progression">
      Progression : {{ reponsesDonnees }} / {{ questions.length }}
      <progress :value="reponsesDonnees" :max="questions.length"></progress>
    </label>

    <section v-if="!termine" class="carte">
      <p class="numero">Question {{ indexQuestion + 1 }} sur {{ questions.length }}</p>
      <img :src="questionCourante.image" :alt="questionCourante.descriptionImage" />
      <h2>{{ questionCourante.texte }}</h2>

      <div class="choix">
        <button
          v-for="(c, i) in questionCourante.choix"
          :key="i"
          :class="classeChoix(i)"
          :disabled="aRepondu"
          @click="repondre(i)"
        >
          {{ symboleChoix(i) }}{{ c }}
        </button>
      </div>

      <p v-if="aRepondu" class="retour" :class="estCorrecte ? 'ok' : 'ko'" role="status">
        <template v-if="estCorrecte">✔ Bonne réponse !</template>
        <template v-else>✘ Mauvaise réponse. La bonne réponse était : {{ questionCourante.choix[questionCourante.bonneReponse] }}.</template>
      </p>

      <button v-if="aRepondu" class="action" @click="suivante">
        {{ derniereQuestion ? 'Voir le résultat' : 'Question suivante' }}
      </button>
    </section>

    <section v-else class="carte resultat">
      <h2>Quiz terminé</h2>
      <p class="final">Votre score final : {{ score }} / {{ questions.length }}</p>
      <button class="action" @click="rejouer">Rejouer</button>
    </section>
  </main>
</template>

<style>
* { box-sizing: border-box; }
body { margin: 0; font-family: 'Segoe UI', Arial, sans-serif; background: #eef1f6; color: #1f2937; }
.quiz { max-width: 560px; margin: 0 auto; padding: 16px; }
.haut { display: flex; justify-content: space-between; align-items: center; gap: 12px; flex-wrap: wrap; }
h1 { margin: 0; font-size: 26px; }
.score { margin: 0; font-weight: 700; font-size: 18px; background: #1d4ed8; color: white; padding: 6px 14px; border-radius: 999px; }
.progression { display: block; margin: 14px 0; font-weight: 600; }
progress { width: 100%; height: 16px; display: block; margin-top: 4px; }
.carte { background: white; border-radius: 12px; padding: 18px; box-shadow: 0 2px 8px rgba(0,0,0,.08); }
.numero { margin: 0 0 8px; color: #4b5563; font-weight: 600; }
img { display: block; width: 100%; max-height: 260px; object-fit: contain; border-radius: 8px; background: #f1f3f5; }
h2 { font-size: 20px; margin: 14px 0; }
.choix { display: grid; gap: 10px; }
.choix button, .action {
  font-size: 17px; padding: 12px; border-radius: 8px; border: 2px solid #1d4ed8;
  background: white; color: #1d4ed8; font-weight: 600; cursor: pointer;
}
.choix button:hover:not(:disabled) { background: #e8efff; }
.choix button:disabled { cursor: not-allowed; opacity: .85; }
.choix .juste { background: #d8f3e0; border-color: #0e5a2a; color: #0e5a2a; opacity: 1; }
.choix .faux { background: #fde2e2; border-color: #8a1313; color: #8a1313; opacity: 1; }
.choix .neutre { border-color: #9ca3af; color: #6b7280; }
.retour { font-weight: 700; margin: 14px 0 8px; }
.retour.ok { color: #0e5a2a; }
.retour.ko { color: #8a1313; }
.action { width: 100%; background: #1d4ed8; color: white; margin-top: 8px; }
.resultat { text-align: center; }
.final { font-size: 24px; font-weight: 700; }
</style>
