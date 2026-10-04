<script setup>
import { computed, ref } from 'vue'

const questions = [
  { texte: 'Quel animal observez-vous sur cette image ?', image: '/images/chien.webp', choix: ['Un chat', 'Un chien', 'Un cheval'], bonneReponse: 1, descriptionImage: 'Un chien' },
  { texte: 'Quel animal est reconnaissable sur cette image ?', image: '/images/chat.jpeg', choix: ['Un chat', 'Un lion', 'Un chameau'], bonneReponse: 0, descriptionImage: 'Un chat' },
  { texte: 'Quel animal possède une longue crinière et est visible ici ?', image: '/images/cheval.jpeg', choix: ['Un chien', 'Un cheval', 'Un lion'], bonneReponse: 1, descriptionImage: 'Un cheval' },
  { texte: 'Quel grand félin reconnaissez-vous sur cette image ?', image: '/images/Lion.jpeg', choix: ['Un lion', 'Un chat', 'Un cheval'], bonneReponse: 0, descriptionImage: 'Un lion' },
  { texte: 'Quel animal du désert est représenté sur cette image ?', image: '/images/chameau.jpeg', choix: ['Un cheval', 'Un chameau', 'Un chien'], bonneReponse: 1, descriptionImage: 'Un chameau' },
]

const indexQuestion = ref(0)
const score = ref(0)
const reponseChoisie = ref(null)
const termine = ref(false)
const questionCourante = computed(() => questions[indexQuestion.value])
const nombreReponses = computed(() => (reponseChoisie.value === null ? indexQuestion.value : indexQuestion.value + 1))
const aRepondu = computed(() => reponseChoisie.value !== null)
const estBonneReponse = computed(() => reponseChoisie.value === questionCourante.value.bonneReponse)

function repondre(indexChoix) {
  if (aRepondu.value || termine.value) return
  reponseChoisie.value = indexChoix
  if (indexChoix === questionCourante.value.bonneReponse) score.value += 1
}

function questionSuivante() {
  if (!aRepondu.value) return
  if (indexQuestion.value === questions.length - 1) {
    termine.value = true
    return
  }
  indexQuestion.value += 1
  reponseChoisie.value = null
}

function rejouer() {
  indexQuestion.value = 0
  score.value = 0
  reponseChoisie.value = null
  termine.value = false
}
</script>

<template>
  <main class="quiz-shell">
    <header class="topbar">
      <div class="brand-mark" aria-hidden="true">Q<span>5</span></div>
      <div><p class="eyebrow">Safari des animaux</p><h1>Quiz visuel</h1></div>
      <div class="score-badge" aria-label="Score actuel"><span>Score</span><strong>{{ score }}<small>/{{ questions.length }}</small></strong></div>
    </header>

    <section v-if="!termine" class="quiz-card" aria-live="polite">
      <div class="progress-header"><span>Question {{ indexQuestion + 1 }} <b>sur {{ questions.length }}</b></span><span>{{ nombreReponses }}/{{ questions.length }} répondue<span v-if="nombreReponses > 1">s</span></span></div>
      <progress :value="nombreReponses" :max="questions.length" aria-label="Progression du quiz"></progress>
      <div class="question-layout">
        <div class="image-frame"><span class="image-label">Observez bien</span><img :src="questionCourante.image" :alt="questionCourante.descriptionImage" /></div>
        <div class="question-content">
          <p class="question-number">0{{ indexQuestion + 1 }}</p>
          <h2>{{ questionCourante.texte }}</h2>
          <div class="answers">
            <button v-for="(choix, index) in questionCourante.choix" :key="choix" type="button" :class="{ selected: reponseChoisie === index, correct: aRepondu && index === questionCourante.bonneReponse, incorrect: aRepondu && reponseChoisie === index && !estBonneReponse }" :disabled="aRepondu" @click="repondre(index)">
              <span class="answer-letter">{{ String.fromCharCode(65 + index) }}</span><span>{{ choix }}</span>
            </button>
          </div>
          <div v-if="aRepondu" class="feedback" :class="estBonneReponse ? 'feedback-good' : 'feedback-bad'" role="status">
            <strong>{{ estBonneReponse ? 'Bonne réponse !' : 'Pas tout à fait.' }}</strong>
            <span>{{ estBonneReponse ? 'Votre score augmente d’un point.' : `La bonne réponse était « ${questionCourante.choix[questionCourante.bonneReponse]} ».` }}</span>
          </div>
          <button v-if="aRepondu" type="button" class="next-button" @click="questionSuivante">{{ indexQuestion === questions.length - 1 ? 'Voir le résultat' : 'Question suivante' }} <span aria-hidden="true">→</span></button>
        </div>
      </div>
    </section>

    <section v-else class="result-card" aria-live="polite">
      <div class="result-icon" aria-hidden="true">✓</div><p class="eyebrow">Quiz terminé</p><h2>Votre regard est affûté.</h2>
      <p class="result-score">{{ score }} <span>/ {{ questions.length }}</span></p>
      <p class="result-message">{{ score === questions.length ? 'Un sans-faute remarquable !' : 'Chaque réponse vous rapproche du monde animal.' }}</p>
      <button type="button" class="restart-button" @click="rejouer">Rejouer <span aria-hidden="true">↻</span></button>
    </section>
    <footer class="footer-note">Cinq images. Une seule bonne réponse.</footer>
  </main>
</template>
