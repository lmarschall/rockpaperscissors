<template>
  <div class="container">
    <div class="row justify-content-md-center">
      <button type="button" class="btn btn-dark" @click="play">Play</button>
    </div>
    <div v-if="run" class="row justify-content-md-center">
      <input type="text" class="form-control" :value="count" />
    </div>
    <div class="row justify-content-md-center">
      <div class="col">
        <Canvas @inputs="handleInputs" />
      </div>
      <div class="col">
        <Opponent :sign="opponent_sign" />
      </div>
    </div>
    <div class="row justify-content-md-center">
      <div class="col">
        <input type="text" class="form-control" :value="player_sign" />
      </div>
      <div class="col">
        <input type="text" class="form-control" :value="opponent_sign" />
      </div>
    </div>
    <div v-if="!run" class="row justify-content-md-center">
      <div class="input-group mb-3">
        <div class="input-group-prepend">
          <span class="input-group-text" id="winner-span">WINNER</span>
        </div>
        <input
          type="text"
          class="form-control"
          aria-describedby="winner-span"
          :value="winner"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.row.justify-content-md-center {
  margin-top: 2vh;
  margin-bottom: 2vh;
}
input.form-control {
  text-align-last: center;
}
</style>

<script setup lang="ts">
import Canvas from "./../components/Canvas.vue";
import Opponent from "./../components/Opponent.vue";

import { onMounted, ref } from "vue";
import ml5 from "ml5";

const run = ref(false);
const opponent_sign = ref("ENEMY");
const player_sign = ref("");
const count = ref(3);
const winner = ref("");

let ready = false;
let interval = null;
let model = null;

onMounted(() => {
  setup();
});

async function setup() {
  // setup own model for detecting custom hand pose
  const options = {
    inputs: 21,
    outputs: 3,
    task: "classification",
    debug: true,
  };
  model = ml5.neuralNetwork(options);
  const modelInfo = {
    model: "/model.json",
    metadata: "/model_meta.json",
    weights: "/model.weights.bin",
  };
  model.load(modelInfo, modelLoaded);
}

async function modelLoaded() {
  console.log("Model Loaded");
  ready = true;
}

async function play() {
  run.value = true;
  opponent_sign.value = "ENEMY";
  interval = window.setInterval(timer, 1000);
}

async function handleInputs(inputs) {
  if (ready) {
    // predict sign
    model.classify(inputs, gotResults);
  }
}

async function gotResults(errors, results) {
  player_sign.value = results[0].label;
}

async function timer() {
  count.value--;
  if (count.value == 0) {
    const signs = ["ROCK", "PAPER", "SCISSORS"];
    const random = Math.floor(Math.random() * signs.length);
    opponent_sign.value = signs[random];
    window.clearInterval(interval);
    run.value = false;
    count.value = 3;
    winner.value = getWinner();
  }
}

function getWinner() {
  if (player_sign.value == "ROCK" && opponent_sign.value == "PAPER") {
    return "OPPONENT";
  }

  if (player_sign.value == "ROCK" && opponent_sign.value == "SCISSORS") {
    return "PLAYER";
  }

  if (player_sign.value == "PAPER" && opponent_sign.value == "ROCK") {
    return "PLAYER";
  }

  if (player_sign.value == "PAPER" && opponent_sign.value == "SCISSORS") {
    return "OPPONENT";
  }

  if (player_sign.value == "SCISSORS" && opponent_sign.value == "ROCK") {
    return "OPPONENT";
  }

  if (player_sign.value == "SCISSORS" && opponent_sign.value == "PAPER") {
    return "PLAYER";
  }
  return "NONE";
}
</script>
