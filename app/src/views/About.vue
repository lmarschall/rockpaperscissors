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
                h: 0,
                landmarks: []
            },
            p5: null,
            model: null,
            video: null
        };
    },
    mounted: function() {
        this.setup();
    },
    methods: {
        setup: function() {
            // setup p5 canvas for drawing hand pose
            this.p5 = new P5(this.sketch);

            // setup ml5 handpose model for detecting hand pose
            const poseNet = ml5.handpose(this.video, this.modelReady);
            poseNet.on("predict", this.gotPose);

            // setup own model for detecting custom hand pose
            let options = {
                input: 2,
                output: 3,
                task: "classification"
            };
            this.model = ml5.neuralNetwork(options);
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
                    // draw hand bounding box
                    p.stroke(0);
                    p.noFill();
                    p.strokeWeight(3);
                    p.rect(self.hand.x, self.hand.y, self.hand.w, self.hand.h);

                    p.stroke("blue");
                    p.strokeWeight(5);

                    // draw hand landmarks
                    for (var i = 0; i < self.hand.landmarks.length; i++) {
                        const landmark = self.hand.landmarks[i];

                        p.point(landmark[0], landmark[1], landmark[2]);
                    }
                }
            };
        },

        modelReady: function() {
            // this.label = this.p5.createDiv("Label: ...");
            // this.confidence = this.p5.createDiv("Confidence: ...");
        },

        gotPose: function(results) {
            // console.log(results);

            this.hand.detected = false;

            // check if anything was detected
            if (results.length) {
                // this.label.html(`Label: ${JSON.stringify(results[0])}`);

                // save the actual hand position to be drawn later
                this.hand.detected = true;
                this.hand.x = results[0].boundingBox.topLeft[0];
                this.hand.y = results[0].boundingBox.bottomRight[1];
                this.hand.w =
                    results[0].boundingBox.bottomRight[0] -
                    results[0].boundingBox.topLeft[0];
                this.hand.h =
                    results[0].boundingBox.topLeft[1] -
                    results[0].boundingBox.bottomRight[1];

                // save hand landmarks
                this.hand.landmarks = results[0].landmarks;

                // set the current pose to be trained
                // const rps = "ROCK"

                // calculate the 3d vectors of each hand annotation

                // define the input of the model data
                let input = {
                    x: 0,
                    y: 0
                };

                // define the output of the model data
                let output = {
                    test: ""
                };

                // add the data to the model
                this.model.addData(input, output);
            }
        }
    }
};
</script>
