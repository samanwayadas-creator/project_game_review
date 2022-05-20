<template>
  <table class="game-board">
    <tr v-for="(row, row_index) in gameBoard" :key="row_index">
      <td v-for="(cell, cell_index) in row" :key="cell_index">
        <transition name="fade" v-if="cell.canFade">
          <TetrominoGenerator v-if="cell.value == '1'" :color="cell.color" />
        </transition>
        <div v-else>
          <TetrominoGenerator v-if="cell.value == '1'" :color="cell.color" />
        </div>
      </td>
    </tr>
  </table>
</template>

<script>
import TetrominoGenerator from "./TetrominoGenerator.vue";
export default {
  name: "GameBoard",

  props: {
    gameBoard: Array,
  },

  components: {
    TetrominoGenerator,
  },

  data() {
    return {
      color: "green",
    };
  },

  methods: {},
};
</script>

<style>
table {
  padding: 0px;
  margin: 0px;
  border-spacing: 0px;
  border-collapse: collapse;
  background-color: white;
}

.game-board {
  border: 10px solid #c9ada1;
  min-width: 340px;
  box-shadow: inset 0 0 10px #464240;
}

td {
  padding: 0px;
  margin: 0px;
  border-spacing: 0;
  width: 2rem;
  height: 2rem;
  border-radius: 10px;
}
</style>
