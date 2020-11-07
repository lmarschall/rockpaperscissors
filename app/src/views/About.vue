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
            hand: {
                detected: false,
                x: 0,
                y: 0,
                w: 0,
                h: 0
            },
            p5: null,
            poseNet: null,
            video: null
        };
    },
    mounted: function() {
        this.setup();
    },
    methods: {
        setup: function() {
            this.p5 = new P5(this.sketch);

            const poseNet = ml5.handpose(this.video, this.modelReady);

            poseNet.on("predict", this.gotPose);
        },

        sketch: function(p) {
            const self = this;

            p.setup = function() {
                p.createCanvas(320, 240);
                self.video = p.createCapture(p.VIDEO, {
                    video: {
                        width: 320,
                        height: 240,
                        aspectRatio: 1
                    }
                });
                self.video.hide();
            };

            p.draw = function() {
                p.image(self.video, 0, 0);
                if (self.hand.detected) {
                    p.rect(self.hand.x, self.hand.y, self.hand.w, self.hand.h);
                }
            };
        },

        modelReady: function() {
            // this.label = this.p5.createDiv("Label: ...");
            // this.confidence = this.p5.createDiv("Confidence: ...");
        },

        gotPose: function(results) {
            console.log(results);

            this.hand.detected = false;

            if (results.length) {
                // this.label.html(`Label: ${JSON.stringify(results[0])}`);

                this.hand.detected = true;
                this.hand.x = results[0].boundingBox.topLeft[0];
                this.hand.y = results[0].boundingBox.bottomRight[1];
                this.hand.w =
                    results[0].boundingBox.bottomRight[0] -
                    results[0].boundingBox.topLeft[0];
                this.hand.h =
                    results[0].boundingBox.topLeft[1] -
                    results[0].boundingBox.bottomRight[1];
            }
        }
    }
};
</script>
