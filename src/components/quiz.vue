<template>
  <div v-if="isQuizReady">
    <div v-if="!showResults">
      <h2>{{ quizQuestions[currentQuestionIndex].question }}</h2>
      <ul>
        <li v-for="(option, index) in quizQuestions[currentQuestionIndex].options" :key="index">
          <button @click="selectAnswer(option.value)">{{ option.text }}</button>
        </li>
      </ul>
    </div>
    <div v-if="showResults">
      <!--img src="/public/assets/{{ result }}.png" alt="Wij denken dat je de {{  result }} eens moet proberen!"-->
      <h2>De {{ result }} lijkt ons wel iets voor jou!</h2>
      <p>{{ description }}</p>
    </div>
  </div>
  <div v-else>Loading...</div>
</template>


<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

// import questions.json function, load as quiz questions in the DOM
const quizQuestions = ref([]);
const rides = ref([]);
const currentQuestionIndex = ref(0);
const isLoading = ref(true);

onMounted(async () => {
  try {
    const response = await axios.get('src/data/questions.json');
    quizQuestions.value = response.data.questions;
    console.log(quizQuestions.value);
  } catch (error) {
    console.error(error);
  }
  finally {
    isLoading.value = false;
  }
});
onMounted(async () => {
  try {
    const responseRides = await axios.get('src/data/rides.json');
    rides.value = responseRides.data;
    console.log(rides.value);
  } catch (error) {
    console.error(error);
  }
  finally {
    isLoading.value = false;
  }
});

//Select Answer code, add value of answer to score total
const selectedAnswer = ref(null);
const score = ref(null);
const matchedRide = ref("none");
const showResults = ref(false);
const result = ref("none");
const description = ref("none");

function selectAnswer(index) {
  if (score.value == null) {
    score.value = 0;
  }
  score.value += index;
  console.log("Score:", score.value);
  nextQuestion();
}

/* Match score total with rides in rides.json
function matchScoreToRide() {

  if (score.value > 15) {
    result.value = "booster";
  }
  else if (score.value > 10) {
    result.value = "reactor";
  }
  else if (score.value > 5) {
    result.value = "botsauto's";
  }
  else {
    result.value = "none";
  }
} */

// The same as above, but with json
function matchScoreToRide() {
  const matchedRide = rides.value.Rides.find(ride => ride.score === score.value); // Access Rides array
  if (matchedRide) {
    result.value = matchedRide.Name;
    description.value = matchedRide.description;
  } else {
    result.value = "none";
  }
}


// Display only current question

function nextQuestion() {
  if (currentQuestionIndex.value < quizQuestions.value.length - 1) {
    currentQuestionIndex.value++;
    console.log("Current question index:", currentQuestionIndex.value)
  } else if (rides.value) {
    matchScoreToRide();
    showResults.value = true;
    console.log("Quiz finished! Result: ", result.value);
  }
  else {
    console.log("whoops")
  }
}

const isQuizReady = computed(() => !isLoading.value && quizQuestions.value.length > 0);
// profit? Maybe add routing to make it an SPA?

</script>
