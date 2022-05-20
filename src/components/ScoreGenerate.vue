<template>
  <div class="score-title" @click="show = !show">score</div>
  <div class="plus">
    {{ currentPoints }}
  </div>
  <transition name="fade">
    <p class="plus" v-if="showPointsToAdd">+ {{ pointsToAdd }}</p>
  </transition>
  <div class="game-over" v-if="gameOver">game over!</div>
  <button class="restart" @click="$emit('restart')" v-if="gameOver">
    Restart!
  </button>
</template>

<script>
export default {
  name: "ScoreGenerate",

  emits: ["restart"],

  props: {
    currentPoints: Number,
    pointsToAdd: Number,
    showPointsToAdd: Boolean,
    gameOver: Boolean,
  },

  data() {
    return {
      show: false,
    };
  },

  computed: {},
};
</script>

<style>
.score-title {
  margin: 1.5rem 0;
}

.score {
  margin: 1.5rem 0;
}

.plus {
  font-family: "Courier New", Courier, monospace;
}

.game-over {
  font-size: 1.5rem;
  margin-top: 1rem;
}

.restart {
  border-radius: 3px;
  background-color: green;
  padding: 10px;
  margin-top: 15px;
}

.restart:hover {
  cursor: pointer;
}
</style>
