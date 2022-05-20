<template>
  <div class="next">
    next
    <div class="tables-container">
      <table
        class="next-table"
        v-for="(each, index) in nextTetrisBlock"
        :key="index"
      >
        <tr v-for="(row, idx) in each.blocksForNext" :key="idx">
          <td v-for="(cell, idx) in row" :key="idx">
            <TetrominoGenerator v-if="cell == '1'" :color="each.color" />
          </td>
        </tr>
      </table>
    </div>
    <ScoreGenerate
      :currentPoints="currentPoints"
      :pointsToAdd="pointsToAdd"
      :showPointsToAdd="showPointsToAdd"
      :gameOver="gameOver"
      @restart="$emit('restart')"
    />
  </div>
</template>

<script>
import TetrominoGenerator from "./TetrominoGenerator.vue";
import ScoreGenerate from "./ScoreGenerate.vue";

export default {
  name: "NextWindow",

  emits: ["restart"],

  components: {
    TetrominoGenerator,
    ScoreGenerate,
  },

  props: {
    nextTetrisBlock: Array,
    currentPoints: Number,
    pointsToAdd: Number,
    showPointsToAdd: Boolean,
    gameOver: Boolean,
  },
};
</script>

<style>
.tables-container {
  margin-top: 10%;
  padding-top: 4%;
  padding-bottom: 2%;
  background-color: azure;
  border-radius: 5px;
  height: 300px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.next-table {
  margin: 10% auto;
}

.next-table td {
  background-color: azure;
}

.next {
  font-family: "Copperplate Gothic Light";
  font-size: 2rem;
  text-align: center;
  padding: 20px;
  min-width: 10rem;
}
</style>
