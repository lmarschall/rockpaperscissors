<template>
    <div id="canvas_container"></div>
</template>

<script>
import ml5 from "ml5";
import P5 from "p5";

export default {
    name: "Canvas",
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
            video: null
        };
    },
    props: {
        modes: {
            collect: false,
            train: false,
            predict: false
        }
    },
    mounted: function() {
        this.setup();
    },
    methods: {
        setup: function() {
            // setup p5 canvas for drawing hand pose
            new P5(this.sketch, "canvas_container");
        },

        loadModel: function() {
            // setup ml5 handpose model for detecting hand pose
            const poseNet = ml5.handpose(this.video, this.modelLoaded);
            poseNet.on("predict", this.gotPose);
        },

        modelLoaded: function() {
            console.log("Hand Pose Model loaded");
        },

        sketch: function(p) {
            const self = this;

            p.setup = function() {
                // p.remove();
                p.createCanvas(320, 240);
                self.video = p.createCapture(p.VIDEO, {
                    video: {
                        width: 320,
                        height: 240,
                        aspectRatio: 1
                    }
                });
                self.video.hide();
                self.loadModel();
            };

            p.draw = function() {
                // mirror the frame and present it in dom
                p.translate(self.video.width, 0);
                p.scale(-1, 1);
                p.image(self.video, 0, 0, self.video.width, self.video.height);

                // check if hand was detected and draw keypoints
                if (self.hand.detected) {
                    // draw hand bounding box
                    p.stroke(0);
                    p.noFill();
                    p.strokeWeight(3);
                    p.rect(self.hand.x, self.hand.y, self.hand.w, self.hand.h);

                    p.stroke("blue");
                    p.strokeWeight(5);

                    // draw hand landmarks
                    for (let i = 0; i < self.hand.landmarks.length; i++) {
                        const landmark = self.hand.landmarks[i];

                        p.point(landmark[0], landmark[1]);
                    }
                }
            };
        },

        gotPose: function(results) {
            // console.log(results);

            this.hand.detected = false;

            // check if anything was detected
            if (results.length) {
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

                this.hand.landmarks = [];

                // save x and y coords of hand 21 landmarks
                for (let i = 0; i < results[0].landmarks.length; i++) {
                    const point = [
                        results[0].landmarks[i][0],
                        results[0].landmarks[i][1]
                    ];
                    this.hand.landmarks.push(point);
                }

                // generate inputs
                let inputs = [];

                for (let i = 0; i < this.hand.landmarks.length; i++) {
                    let x = this.hand.landmarks[i][0] / 320;
                    let y = this.hand.landmarks[i][1] / 240;
                    inputs.push(x);
                    inputs.push(y);
                }

                this.$emit("inputs", inputs);
            }
        }
    }
};
</script>
