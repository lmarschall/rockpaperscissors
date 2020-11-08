<template>
    <div class="container">
        <div class="row justify-content-md-center">
            <button type="button" class="btn btn-dark" v-on:click="play">Play</button>
        </div>
        <div v-if="run" class="row justify-content-md-center">
            <input type="text" class="form-control" v-bind:value="count" />
        </div>
        <div class="row justify-content-md-center">
            <div class="col">
                <Canvas v-on:inputs="handleInputs" />
            </div>
            <div class="col">
                <Opponent v-bind:sign="opponent_sign" />
            </div>
        </div>
        <div class="row justify-content-md-center">
            <div class="col">
                <input type="text" class="form-control" v-bind:value="player_sign" />
            </div>
            <div class="col">
                <input type="text" class="form-control" v-bind:value="opponent_sign" />
            </div>
        </div>
        <div v-if="!run" class="row justify-content-md-center">
            <div class="input-group mb-3">
                <div class="input-group-prepend">
                    <span class="input-group-text" id="winner-span">WINNER</span>
                </div>
                <input type="text" class="form-control" aria-describedby="winner-span" v-bind:value="winner" />
            </div>
        </div>
    </div>
</template>

<style scoped>
.row.justify-content-md-center {
    margin-top: 2vh;
    margin-bottom: 2vh;
}
input.form-control {
    text-align-last: center;
}
</style>

<script>
import Canvas from "@/components/Canvas.vue";
import Opponent from "@/components/Opponent.vue";

import ml5 from "ml5";

export default {
    name: "Game",
    components: {
        Canvas,
        Opponent
    },
    data() {
        return {
            ready: false,
            run: false,
            interval: null,
            count: 3,
            player_sign: "",
            opponent_sign: "ENEMY",
            winner: ""
        };
    },
    mounted: function() {
        this.setup();
    },
    methods: {
        play: function() {
            this.run = true;
            this.opponent_sign = "ENEMY";
            this.interval = window.setInterval(this.timer, 1000);
        },

        timer: function() {
            this.count--;
            if (this.count == 0) {
                const signs = ["ROCK", "PAPER", "SCISSORS"];
                const random = Math.floor(Math.random() * signs.length);
                this.opponent_sign = signs[random];
                window.clearInterval(this.interval);
                this.run = false;
                this.count = 3;
                this.winner = this.getWinner();
            }
        },

        getWinner: function() {
            if (this.player_sign == "ROCK" && this.opponent_sign == "PAPER") {
                return "OPPONENT";
            }

            if (
                this.player_sign == "ROCK" &&
                this.opponent_sign == "SCISSORS"
            ) {
                return "PLAYER";
            }

            if (this.player_sign == "PAPER" && this.opponent_sign == "ROCK") {
                return "PLAYER";
            }

            if (
                this.player_sign == "PAPER" &&
                this.opponent_sign == "SCISSORS"
            ) {
                return "OPPONENT";
            }

            if (
                this.player_sign == "SCISSORS" &&
                this.opponent_sign == "ROCK"
            ) {
                return "OPPONENT";
            }

            if (
                this.player_sign == "SCISSORS" &&
                this.opponent_sign == "PAPER"
            ) {
                return "PLAYER";
            }
            return "NONE";
        },

        setup: function() {
            // setup game

            // setup own model for detecting custom hand pose
            let options = {
                inputs: 21,
                outputs: 3,
                task: "classification",
                debug: true
            };
            this.model = ml5.neuralNetwork(options);
            const modelInfo = {
                model: "/model.json",
                metadata: "/model_meta.json",
                weights: "/model.weights.bin"
            };
            this.model.load(modelInfo, this.modelLoaded);
        },

        modelLoaded: function() {
            console.log("Model Loaded");
            this.ready = true;
        },

        handleInputs: function(inputs) {
            if (this.ready) {
                // predict sign
                this.model.classify(inputs, this.gotResults);
            }
        },

        gotResults: function(errors, results) {
            this.player_sign = results[0].label;
            // console.log(this.player_sign);
        }
    }
};
</script>
