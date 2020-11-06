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
            poseNet: null,
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

            this.poseNet = ml5.handpose(this.video, this.modelReady);

            this.poseNet.on("predict", this.gotPose);
        },

        modelReady: function() {
            this.label = this.p5.createDiv("Label: ...");
            this.confidence = this.p5.createDiv("Confidence: ...");
        },

        gotPose: function(results) {
            // if (error) {
            //     console.error(error);
            // }

            console.log(results);

            if (results.length) {
                this.label.html(`Label: ${JSON.stringify(results[0])}`);
            }

            // The results are in an array ordered by confidence.
            // Show the first label and confidence
            //
            // Round the confidence to 0.01
            // this.confidence.html(
            //     `Confidence: ${this.p5.nf(results[0].confidence, 0, 2)}`
            // );
        }
    }
};
</script>
