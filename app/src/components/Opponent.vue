<template>
    <div id="opponent_container"></div>
</template>

<script>
import P5 from "p5";

export default {
    name: "Opponent",
    props: {
        sign: {
            type: String,
            required: true,
            default: ""
        },

        width: {
            type: Number,
            required: true,
            default: 320
        },

        height: {
            type: Number,
            required: true,
            default: 240
        }
    },
    mounted: function() {
        this.setup();
    },
    methods: {
        setup: function() {
            // setup p5 canvas for drawing hand pose
            new P5(this.sketch, "opponent_container");
        },

        sketch: function(p) {
            const self = this;

            let rock, paper, scissors, enemy;

            p.preload = function() {
                rock = p.loadImage("/rock.png");
                paper = p.loadImage("/paper.png");
                scissors = p.loadImage("/scissors.png");
                enemy = p.loadImage("/enemy.png");
            };

            p.setup = function() {
                p.createCanvas(self.width, self.height);
            };

            p.draw = function() {
                p.clear();
                switch (self.sign) {
                    case "ROCK":
                        p.image(rock, 0, 0, self.width, self.height);
                        break;
                    case "PAPER":
                        p.image(paper, 0, 0, self.width, self.height);
                        break;
                    case "SCISSORS":
                        p.image(scissors, 0, 0, self.width, self.height);
                        break;
                    default:
                        p.image(enemy, 0, 0, self.width, self.height);
                        break;
                }
            };
        }
    }
};
</script>
