<template>
  <div id="canvas_container"></div>
</template>

<script setup lang="ts">
import ml5 from "ml5";
import P5 from "p5";
import { onMounted } from "vue";

const emit = defineEmits(["inputs"]);

const hand = {
  detected: false,
  x: 0,
  y: 0,
  w: 0,
  h: 0,
  landmarks: [],
};

let video = null;

onMounted(() => {
  setup();
});

async function setup() {
  new P5(sketch, "canvas_container");
}

async function loadModel() {
  // setup ml5 handpose model for detecting hand pose
  const poseNet = ml5.handpose(video, modelLoaded);
  poseNet.on("predict", gotPose);
}

async function modelLoaded() {
  console.log("Hand Pose Model loaded");
}

async function sketch(p) {
  p.setup = function () {
    // p.remove();
    p.createCanvas(320, 240);
    video = p.createCapture(p.VIDEO, {
      video: {
        width: 320,
        height: 240,
        aspectRatio: 1,
      },
    });
    video.hide();
    loadModel();
  };

  p.draw = function () {
    // mirror the frame and present it in dom
    p.translate(video.width, 0);
    p.scale(-1, 1);
    p.image(video, 0, 0, video.width, video.height);

    // check if hand was detected and draw keypoints
    if (hand.detected) {
      // draw hand bounding box
      p.stroke(0);
      p.noFill();
      p.strokeWeight(3);
      p.rect(hand.x, hand.y, hand.w, hand.h);

      p.stroke("blue");
      p.strokeWeight(5);

      // draw hand landmarks
      for (let i = 0; i < hand.landmarks.length; i++) {
        const landmark = hand.landmarks[i];

        p.point(landmark[0], landmark[1]);
      }
    }
  };
}

async function gotPose(results) {
  hand.detected = false;

  // check if anything was detected
  if (results.length) {
    // save the actual hand position to be drawn later
    hand.detected = true;
    hand.x = results[0].boundingBox.topLeft[0];
    hand.y = results[0].boundingBox.bottomRight[1];
    hand.w =
      results[0].boundingBox.bottomRight[0] - results[0].boundingBox.topLeft[0];
    hand.h =
      results[0].boundingBox.topLeft[1] - results[0].boundingBox.bottomRight[1];

    hand.landmarks = [];

    // save x and y coords of hand 21 landmarks
    for (let i = 0; i < results[0].landmarks.length; i++) {
      const point = [results[0].landmarks[i][0], results[0].landmarks[i][1]];
      hand.landmarks.push(point);
    }

    // generate inputs
    const inputs = [];

    for (let i = 0; i < hand.landmarks.length; i++) {
      const x = hand.landmarks[i][0] / 320;
      const y = hand.landmarks[i][1] / 240;
      inputs.push(x);
      inputs.push(y);
    }

    emit("inputs", inputs);
  }
}
</script>
