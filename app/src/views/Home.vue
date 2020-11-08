<template>
    <div class="home">
        <h1>Rock Paper Scissors</h1>
        <h3>powered by vue.js, ml5.js and p5.js</h3>
        <div class="d-flex flex-row justify-content-center">
            <div class="card flex-fill">
                <img class="img-fluid" src="/rock.png" alt="...">
            </div>
            <div class="card flex-fill">
                <img class="img-fluid" src="/paper.png" alt="...">
            </div>
            <div class="card  flex-fill">
                <img class="img-fluid" src="/scissors.png" alt="...">
            </div>
        </div>

        <!-- <img alt="Vue logo" src="../assets/logo.png" /> -->
        <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
        <Game />
    </div>
</template>

<style scoped>
h1 {
    margin-top: 2vh;
}
.card.flex-fill {
    margin: 2rem;
    max-width: 10rem;
}
</style>

<script>
// @ is an alias to /src
// import HelloWorld from "@/components/HelloWorld.vue";
import Game from "@/components/Game.vue";

export default {
    name: "Home",
    components: {
        // HelloWorld,
        Game
    }
};
</script>
