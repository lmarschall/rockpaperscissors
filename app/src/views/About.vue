<template>
    <div class="about">
        <h1>This is an about page</h1>
    </div>
</template>

<script>
import ml5 from "ml5";
import P5 from "p5";

export default {
    name: "About",
    data() {
        return {
            classifier: null,
            video: null,
            label: null,
            confidence: null
        };
    },
    mounted: function() {
        // this.preload();
        // this.setup();
        const P5 = require('p5');
        console.log(P5);
        let test = new P5();
        this.video = test.createCapture(test.VIDEO, {
              video: {
                  width: 160,
                  height: 120,
                  aspectRatio: 1
              }
          });
    },
    methods: {
      preload: function() {
          // Create a camera input
          this.video = P5.createCapture(P5.VIDEO, {
              video: {
                  width: 160,
                  height: 120,
                  aspectRatio: 1
              }
          });
          //   video = createCapture(VIDEO, video_frame);
          // Load the DoodleNet Image Classification model
          this.classifier = ml5.imageClassifier("DoodleNet", this.video);
      },

      setup: function() {
          // Create a 'label' and 'confidence' div to hold results
          this.label = P5.createDiv("Label: ...");
          this.confidence = P5.createDiv("Confidence: ...");
          // input = document.getElementById("data_transfer");

          this.classifyVideo();
      },

      // Get a prediction for the current video frame
      classifyVideo: function() {
          this.classifier.classify(this.gotResult);
      },

      // A function to run when we get any errors and the results
      gotResult: function(error, results) {
          // Display error in the console
          if (error) {
              console.error(error);
          }
          // The results are in an array ordered by confidence.
          // console.log(results);
          // Show the first label and confidence
          this.label.html(`Label: ${results[0].label}`);
          this.confidence.html(
              `Confidence: ${ml5.nf(results[0].confidence, 0, 2)}`
          ); // Round the confidence to 0.01
          // console.log(input);
          // input.value = results[0].label;
          // document.label = results[0].label;
          // Call classifyVideo again
          this.classifyVideo();
      }
    }
};
</script>
