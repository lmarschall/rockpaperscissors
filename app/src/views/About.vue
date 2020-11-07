<template>
    <div class="about">
        <h1>This is an about page</h1>
        <select name="signs" v-model="sign" id="signs">
            <option disabled value="">SIGN</option>
            <option v-for="sign in signs" v-bind:key="sign">{{ sign }}</option>
        </select>
        <button
            type="button"
            v-on:click="
                modes.collect = true;
                collectCounter = 100;
            "
        >
            Collect
        </button>
        <button type="button" v-on:click="save">Save</button>
        <button type="button" v-on:click="dataReady">Train</button>
        <button type="button" v-on:click="modes.predict = !modes.predict">
            Predict
        </button>
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
                // vectors: []
            },
            sign: "",
            signs: ["ROCK", "PAPER", "SCISSORS"],
            modes: {
                collect: false,
                train: false,
                predict: false
            },
            collectCounter: 0,
            p5: null,
            model: null,
            video: null,
            label: null
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
                inputs: 21,
                outputs: 3,
                task: "classification",
                debug: true
            };
            this.model = ml5.neuralNetwork(options);
            // const modelInfo = {
            //     model: "/model.json",
            //     metadata: "/model_meta.json",
            //     weights: "/model.weights.bin"
            // }
            // this.model.load(modelInfo, this.modelLoaded);
        },

        modalLoaded: function() {
            console.log("Model loaded");
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
                // draw an arrow for a vector at a given base position
                // function drawArrow(base, vec, myColor) {
                //     p.push();
                //     p.stroke(myColor);
                //     p.strokeWeight(3);
                //     p.fill(myColor);
                //     p.translate(base.x, base.y);
                //     p.line(0, 0, vec.x, vec.y);
                //     p.rotate(vec.heading());
                //     let arrowSize = 7;
                //     p.translate(vec.mag() - arrowSize, 0);
                //     p.triangle(0, arrowSize / 2, 0, -arrowSize / 2, arrowSize, 0);
                //     p.pop();
                // }

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

                    // console.log(self.hand.landmarks);

                    // draw hand landmarks
                    for (let i = 0; i < self.hand.landmarks.length; i++) {
                        const landmark = self.hand.landmarks[i];

                        p.point(landmark[0], landmark[1]);
                    }

                    // draw hand lines for indexFinder, only, test function
                    // for (let i = 0; i < self.hand.vectors.length; i++) {

                    //     if(i<(self.hand.vectors.length-1)) {
                    //         const base = self.hand.vectors[i]
                    //         const next = self.hand.vectors[i+1];
                    //         // const v0 = p.createVector(0, 0, 0);
                    //         const v1 = p.createVector(base[0], base[1]);
                    //         const v2 = p.createVector(next[0], next[1]);
                    //         // p.line(base[0], base[1], next[0], next[1])
                    //         const diff = v2.sub(v1);
                    //         diff.normalize();

                    //         drawArrow(v1, diff, 'blue');
                    //         // drawArrow(v0, v2, 'green');
                    //         // drawArrow(v1, v2, 'purple');
                    //     }
                    // }
                }
            };
        },

        modelReady: function() {
            this.label = this.p5.createDiv("Label: ...");
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

                // check for collecting
                if (
                    this.modes.collect &&
                    this.collectCounter > 0 &&
                    this.sign != ""
                ) {
                    // save data to model

                    let target = [this.sign];
                    this.model.addData(inputs, target);

                    console.log("collecting");
                    console.log(this.collectCounter);
                    console.log(this.sign);

                    // decrease collectcounter
                    this.collectCounter--;
                }

                // check for predicting
                if (this.modes.predict) {
                    // predict sign
                    this.model.classify(inputs, this.gotResults);
                }
            }
            if (this.hand.detected) {
                // this.label.html(`Label: ${JSON.stringify(results[0])}`);
                this.label.html(`Label: Hand detected!`);
            } else {
                this.label.html("No Hand detected!");
            }
        },

        gotResults: function(erros, results) {
            console.log("PREDICTIONS");
            console.log(results);
        },

        save: function() {
            console.log("Save Data");
            this.model.saveData();
        },

        train: function() {
            console.log("start training");
            this.model.loadData("/hands_signs_data.json", this.dataReady);
        },

        dataReady: function() {
            console.log("data ready");
            // this.model.normalizeData();
            this.model.train({ epochs: 20 }, this.finished);
        },

        finished: function() {
            console.log("training finished");
            this.model.save();
        }
    }
};
</script>
