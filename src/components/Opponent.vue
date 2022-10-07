<template>
  <div id="opponent_container"></div>
</template>

<script setup lang="ts">
import P5 from "p5";
import { onMounted } from "@vue/runtime-core";

const props = defineProps({
  sign: { type: String, required: true, default: "" },
  width: { type: Number, required: true, default: 320 },
  height: { type: Number, required: true, default: 240 },
});

onMounted(() => {
  setup();
});

async function setup() {
  // setup p5 canvas for drawing hand pose
  new P5(sketch, "opponent_container");
}

async function sketch(p) {
  let rock, paper, scissors, enemy;

  p.preload = function () {
    rock = p.loadImage("/rock.png");
    paper = p.loadImage("/paper.png");
    scissors = p.loadImage("/scissors.png");
    enemy = p.loadImage("/enemy.png");
  };

  p.setup = function () {
    p.createCanvas(props.width, props.height);
  };

  p.draw = function () {
    p.clear();
    switch (props.sign) {
      case "ROCK":
        p.image(rock, 0, 0, props.width, props.height);
        break;
      case "PAPER":
        p.image(paper, 0, 0, props.width, props.height);
        break;
      case "SCISSORS":
        p.image(scissors, 0, 0, props.width, props.height);
        break;
      default:
        p.image(enemy, 0, 0, props.width, props.height);
        break;
    }
  };
}
</script>
