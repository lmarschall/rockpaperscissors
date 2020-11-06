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
            p5: null,
            classifier: null,
            video: null,
            label: null,
            confidence: null
        };
    },
    mounted: function() {
        this.p5 = new P5();
        this.preload();
    },
    methods: {
        preload: function() {
            // Create a camera input
            this.video = this.p5.createCapture(this.p5.VIDEO, {
                video: {
                    width: 160,
                    height: 120,
                    aspectRatio: 1
                }
            });
            //   video = createCapture(VIDEO, video_frame);

            // Load the DoodleNet Image Classification model
            this.classifier = ml5.imageClassifier(
                "DoodleNet",
                this.video,
                this.setup
            );
        },

        setup: function() {
            // Create a 'label' and 'confidence' div to hold results
            this.label = this.p5.createDiv("Label: ...");
            this.confidence = this.p5.createDiv("Confidence: ...");

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
            // Show the first label and confidence
            this.label.html(`Label: ${results[0].label}`);
            // Round the confidence to 0.01
            this.confidence.html(
                `Confidence: ${this.p5.nf(results[0].confidence, 0, 2)}`
            );

            // Call classifyVideo again
            this.classifyVideo();
        }
    }
};
</script>
