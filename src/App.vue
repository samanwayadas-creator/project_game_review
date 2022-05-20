<template>
  <KeyMapped
    @rightLeftFunc="rightLeftFunc"
    @rotateFunc="rotateFunc"
    @downFunc="downFunc"
  />
  <GameBoard :gameBoard="gameBoard" />
  <NextWindow
    :nextTetrisBlock="nextTetrisBlock"
    :currentPoints="currentPoints"
    :pointsToAdd="pointsToAdd"
    :showPointsToAdd="showPointsToAdd"
    :gameOver="gameOver"
    @restart="restart"
  />
</template>

<script>
import NextWindow from "./components/NextWindow.vue";
import KeyMapped from "./components/KeyMapped.vue";
import GameBoard from "./components/GameBoard.vue";

const ROWS = 20;
const COLS = 10;

export default {
  name: "App",
  components: {
    NextWindow,
    KeyMapped,
    GameBoard,
  },

  data() {
    return {
      gameBoard: [],
      tetrominoType: 0,
      leftRightTurnTry: true,
      dropButtonPressed: false,
      nextTetrisBlock: [],
      currentPoints: 0,
      pointsToAdd: 0,
      showPointsToAdd: false,
      gameOver: false,
    };
  },

  methods: {
    restart() {
      this.gameBoard = [];
      this.gameOver = false;
      this.nextTetrisBlock = [];
      this.currentPoints = 0;
      this.pointsToAdd = 0;
      this.generategameBoard();
      this.generateTetromino(true);
      this.startGame();
    },

    startGame() {
      let printDelayed = setInterval(() => {
        if (this.dropButtonPressed && this.findActiveCell()) {
          this.dropButtonPressed = false;
        } else if (!this.downFunc(false)) {
          clearInterval(printDelayed); // this stops the loop
          if (this.findFullRows()) {
            setTimeout(() => {
              this.checkLines();
              this.leftRightTurnTry = false;
              setTimeout(() => {
                if (this.generateTetromino(false)) this.startGame();
                else clearInterval(printDelayed);
              }, 400);
            }, 400);
          } else {
            if (this.generateTetromino(false)) this.startGame();
            else clearInterval(printDelayed);
          }
        }
      }, 1000);
    },

    findActiveCell() {
      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (
            this.gameBoard[i][j].value === "1" &&
            this.gameBoard[i][j].cellActiveStatus
          ) {
            return true;
          }
        }
      }
      return false;
    },

    findFullRows() {
      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (this.gameBoard[i][j].value === "0") break;
          if (
            j === this.gameBoard[i].length - 1 &&
            this.gameBoard[i][j].value === "1"
          ) {
            return true;
          }
        }
      }
      return false;
    },

    rotateFunc() {
      if (!this.leftRightTurnTry) return;

      let lastValues = [];
      let color = "";

      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (
            this.gameBoard[i][j].value === "1" &&
            this.gameBoard[i][j].cellActiveStatus
          ) {
            lastValues.push({
              i: i,
              j: j,
            });
            color = this.gameBoard[i][j].color;
          }
        }
      }
      if (lastValues.length === 0) return;

      let maxRight = 0;
      let minLeft = lastValues[0].j;
      let top = lastValues[0].i;
      let bottom = 0;

      for (let i = 0; i < lastValues.length; i++) {
        if (lastValues[i].j > maxRight) {
          maxRight = lastValues[i].j;
        }
        if (lastValues[i].j < minLeft) {
          minLeft = lastValues[i].j;
        }
        if (lastValues[i].i < top) {
          top = lastValues[i].i;
        }
        if (lastValues[i].i > bottom) {
          bottom = lastValues[i].i;
        }
      }

      let rowLength = maxRight - minLeft;
      let colLength = bottom - top;

      let rowToAdd = 0;
      let colToAdd = 0;
      if (colLength - rowLength < 0) {
        rowToAdd = colLength - rowLength;
      } else if (colLength - rowLength > 0) {
        colToAdd = colLength - rowLength;
      }

      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (
            this.gameBoard[i][j].value === "1" &&
            this.gameBoard[i][j].cellActiveStatus
          ) {
            this.gameBoard[i][j].value = "0";
            this.gameBoard[i][j].cellActiveStatus = false;
          }
        }
      }

      //
      let index = 0;
      let i_from_0 = 0;
      let j_from_0 = 0;
      let correction_j = top;
      let correction_i = minLeft;

      let pushToRight = 0;
      let pushToTop = 0;

      //should be pushed
      for (let i = top + rowToAdd; i <= bottom; i++) {
        j_from_0 = 0;
        for (let j = minLeft; j <= maxRight + colToAdd; j++) {
          if (i === lastValues[index].i && j === lastValues[index].j) {
            if (0 > correction_i + (rowLength - i_from_0)) {
              pushToRight = -(correction_i + (rowLength - i_from_0));
            }
            if (correction_j + j_from_0 >= ROWS) {
              pushToTop--;
            }
            index < lastValues.length - 1 ? index++ : null;
          }
          j_from_0++;
        }
        i_from_0++;
      }

      if (
        (this.tetrominoType === 0 ||
          this.tetrominoType === 1 ||
          this.tetrominoType === 2) &&
        rowLength === 2
      ) {
        pushToRight++;
      } else if (this.tetrominoType === 4 && rowLength === 3) {
        pushToRight += 3;
      } else if (
        this.tetrominoType >= 5 &&
        rowLength === 1 &&
        maxRight != COLS - 1
      ) {
        pushToRight++;
      }

      index = 0;
      i_from_0 = 0;
      let canTurn = true;
      for (let i = top + rowToAdd; i <= bottom; i++) {
        j_from_0 = 0;
        for (let j = minLeft; j <= maxRight + colToAdd; j++) {
          if (i === lastValues[index].i && j === lastValues[index].j) {
            if (
              pushToTop + correction_j + j_from_0 > ROWS ||
              pushToRight + correction_i + (rowLength - i_from_0) > COLS
            ) {
              canTurn = false;
            } else if (
              this.gameBoard[pushToTop + correction_j + j_from_0][
                pushToRight + correction_i + (rowLength - i_from_0)
              ].value === "1"
            ) {
              canTurn = false;
            }
            index < lastValues.length - 1 ? index++ : null;
          }
          j_from_0++;
        }
        i_from_0++;
      }

      if (canTurn) {
        index = 0;
        i_from_0 = 0;
        for (let i = top + rowToAdd; i <= bottom; i++) {
          j_from_0 = 0;
          for (let j = minLeft; j <= maxRight + colToAdd; j++) {
            if (i === lastValues[index].i && j === lastValues[index].j) {
              this.gameBoard[pushToTop + correction_j + j_from_0][
                pushToRight + correction_i + (rowLength - i_from_0)
              ].value = "1";
              this.gameBoard[pushToTop + correction_j + j_from_0][
                pushToRight + correction_i + (rowLength - i_from_0)
              ].cellActiveStatus = true;
              this.gameBoard[pushToTop + correction_j + j_from_0][
                pushToRight + correction_i + (rowLength - i_from_0)
              ].color = color;
              index < lastValues.length - 1 ? index++ : null;
            }
            j_from_0++;
          }
          i_from_0++;
        }
      } else {
        for (let i = 0; i < lastValues.length; i++) {
          this.gameBoard[lastValues[i].i][lastValues[i].j].value = "1";
          this.gameBoard[lastValues[i].i][
            lastValues[i].j
          ].cellActiveStatus = true;
          this.gameBoard[lastValues[i].i][lastValues[i].j].color = color;
        }
      }
    },

    rightLeftFunc(direction) {
      if (!this.leftRightTurnTry) return;
      let lastValues = [];
      let dropTryMore = true;
      let color = "";

      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (
            this.gameBoard[i][j].value === "1" &&
            this.gameBoard[i][j].cellActiveStatus
          ) {
            lastValues.push({
              i: i,
              j: j,
            });
            color = this.gameBoard[i][j].color;
          }
        }
      }

      if (lastValues.length === 0) return;

      //max search on the right
      let maxIdxRight = 0;
      let maxRight = 0;
      let minLeft = lastValues[0].j;
      let minIdxLeft = 0;

      //max search on the right

      for (let i = 0; i < lastValues.length; i++) {
        if (lastValues[i].j > maxRight) {
          maxRight = lastValues[i].j;
          maxIdxRight = i;
        }
        if (lastValues[i].j < minLeft) {
          minLeft = lastValues[i].j;
          minIdxLeft = i;
        }
      }

      if (
        (lastValues[maxIdxRight].j + 1 > COLS - 1 && direction === "right") ||
        (lastValues[minIdxLeft].j === 0 && direction === "left")
      ) {
        dropTryMore = false;
      } else {
        for (let i = 0; i < lastValues.length; i++) {
          if (
            direction === "right" &&
            this.gameBoard[lastValues[i].i][lastValues[i].j + 1].value ===
              "1" &&
            !this.gameBoard[lastValues[i].i][lastValues[i].j + 1]
              .cellActiveStatus
          ) {
            dropTryMore = false;
          } else if (
            direction === "left" &&
            this.gameBoard[lastValues[i].i][lastValues[i].j - 1].value ===
              "1" &&
            !this.gameBoard[lastValues[i].i][lastValues[i].j - 1]
              .cellActiveStatus
          ) {
            dropTryMore = false;
          }
        }
      }

      if (dropTryMore) {
        for (let i = 0; i < this.gameBoard.length; i++) {
          for (let j = 0; j < this.gameBoard[i].length; j++) {
            if (
              this.gameBoard[i][j].value === "1" &&
              this.gameBoard[i][j].cellActiveStatus
            ) {
              this.gameBoard[i][j].value = "0";
              this.gameBoard[i][j].cellActiveStatus = false;
            }
          }
        }

        let index = 0;
        for (let i = 0; i < this.gameBoard.length; i++) {
          for (let j = 0; j < this.gameBoard[i].length; j++) {
            if (
              lastValues[index].i === i &&
              lastValues[index].j === j &&
              direction === "right"
            ) {
              this.gameBoard[i][j + 1].value = "1";
              this.gameBoard[i][j + 1].cellActiveStatus = true;
              this.gameBoard[i][j + 1].color = color;
              index < lastValues.length - 1 ? index++ : null;
            } else if (
              lastValues[index].i === i &&
              lastValues[index].j === j &&
              direction === "left"
            ) {
              this.gameBoard[i][j - 1].value = "1";
              this.gameBoard[i][j - 1].cellActiveStatus = true;
              this.gameBoard[i][j - 1].color = color;
              index < lastValues.length - 1 ? index++ : null;
            }
          }
        }
        return true;
      } else {
        return false;
      }
    },

    checkLines() {
      let indexOfFullRows = [];

      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (this.gameBoard[i][j].value === "0") break;
          if (
            j === this.gameBoard[i].length - 1 &&
            this.gameBoard[i][j].value === "1"
          ) {
            for (let k = 0; k < this.gameBoard[i].length; k++) {
              this.gameBoard[i][k].value = "0";
            }
            indexOfFullRows.push(i);
          }
        }
      }

      if (indexOfFullRows.length > 0) {
        this.showPointsToAdd = true;

        if (indexOfFullRows.length > 1) {
          this.pointsToAdd = 100 * indexOfFullRows.length;
          this.currentPoints += this.pointsToAdd;
        } else {
          this.pointsToAdd = 100;
          this.currentPoints += this.pointsToAdd;
        }
      }

      setTimeout(() => {
        if (indexOfFullRows.length > 0) {
          for (let h = 0; h < indexOfFullRows.length; h++) {
            let lastValues = [];
            for (let i = 0; i <= indexOfFullRows[h]; i++) {
              for (let j = 0; j < this.gameBoard[i].length; j++) {
                if (this.gameBoard[i][j].value === "1") {
                  lastValues.push({
                    i: i,
                    j: j,
                    color: this.gameBoard[i][j].color,
                  });
                  this.gameBoard[i][j].value = "0";
                }
              }
            }
            let index = 0;
            for (let i = 0; i <= indexOfFullRows[h]; i++) {
              for (let j = 0; j < this.gameBoard[i].length; j++) {
                if (lastValues[index].i === i && lastValues[index].j === j) {
                  this.gameBoard[lastValues[index].i + 1][j].value = "1";
                  this.gameBoard[lastValues[index].i + 1][j].canFade = true;
                  this.gameBoard[lastValues[index].i + 1][j].color =
                    lastValues[index].color;
                  index === lastValues.length - 1 ? null : index++;
                }
              }
            }

            for (let i = 0; i <= indexOfFullRows[h]; i++) {
              for (let j = 0; j < this.gameBoard[i].length; j++) {
                if (this.gameBoard[i][j].value === "0") {
                  this.gameBoard[i][j].canFade = false;
                }
              }
            }
          }

          this.showPointsToAdd = false;
        }
      }, 400);

      if (indexOfFullRows.length > 0) return true;
      return false;
    },

    downFunc(isPressed) {
      if (!this.leftRightTurnTry) return;
      isPressed
        ? (this.dropButtonPressed = true)
        : (this.dropButtonPressed = false);
      let lastValues = [];
      let dropTryMore = true;
      let color = "";

      for (let i = 0; i < this.gameBoard.length; i++) {
        for (let j = 0; j < this.gameBoard[i].length; j++) {
          if (
            this.gameBoard[i][j].value === "1" &&
            this.gameBoard[i][j].cellActiveStatus
          ) {
            lastValues.push({
              i: i,
              j: j,
            });
            color = this.gameBoard[i][j].color;
          }
        }
      }

      let maxIdx = 0;
      let max = 0;
      for (let i = 0; i < lastValues.length; i++) {
        if (lastValues[i].i > max) {
          max = lastValues[i].i;
          maxIdx = i;
        }
      }

      if (lastValues.length === 0 || lastValues[maxIdx].i + 1 > ROWS - 1) {
        dropTryMore = false;
      } else {
        for (let i = 0; i < lastValues.length; i++) {
          if (
            !this.gameBoard[lastValues[i].i + 1][lastValues[i].j]
              .cellActiveStatus &&
            this.gameBoard[lastValues[i].i + 1][lastValues[i].j].value === "1"
          ) {
            dropTryMore = false;
          }
        }
      }

      if (dropTryMore) {
        for (let i = 0; i < this.gameBoard.length; i++) {
          for (let j = 0; j < this.gameBoard[i].length; j++) {
            if (
              this.gameBoard[i][j].value === "1" &&
              this.gameBoard[i][j].cellActiveStatus
            ) {
              this.gameBoard[i][j].value = "0";
              this.gameBoard[i][j].cellActiveStatus = false;
            }
          }
        }

        let index = 0;
        for (let i = 0; i < this.gameBoard.length; i++) {
          for (let j = 0; j < this.gameBoard[i].length; j++) {
            if (lastValues[index].i === i && lastValues[index].j === j) {
              this.gameBoard[i + 1][j].value = "1";
              this.gameBoard[i + 1][j].cellActiveStatus = true;
              this.gameBoard[i + 1][j].color = color;
              index < lastValues.length - 1 ? index++ : null;
            }
          }
        }
        return true;
      } else if (!dropTryMore && lastValues.length > 0) {
        for (let i = 0; i < lastValues.length; i++) {
          this.gameBoard[lastValues[i].i][
            lastValues[i].j
          ].cellActiveStatus = false;
          this.gameBoard[lastValues[i].i][lastValues[i].j].canFade = true;
        }
        return false;
      }
    },

    generateTetromino(gameStart) {
      this.leftRightTurnTry = true;

      let colors = [
        "blue",
        "green",
        "lightblue",
        "orange",
        "purple",
        "red",
        "yellow",
      ];
      let tetrominos = [
        [
          ["0", "1", "1", "1"],
          ["0", "0", "0", "1"],
        ],
        [
          ["0", "1", "1", "1"],
          ["0", "1", "0", "0"],
        ],
        [
          ["0", "0", "1", "0"],
          ["0", "1", "1", "1"],
        ],
        [
          ["0", "0", "1", "1"],
          ["0", "0", "1", "1"],
        ],
        [
          ["1", "1", "1", "1"],
          ["0", "0", "0", "0"],
        ],
        [
          ["0", "0", "1", "1"],
          ["0", "1", "1", "0"],
        ],
        [
          ["0", "1", "1", "0"],
          ["0", "0", "1", "1"],
        ],
      ];

      let tetrominosForNext = [
        [
          ["1", "1", "1"],
          ["0", "0", "1"],
        ],
        [
          ["1", "1", "1"],
          ["1", "0", "0"],
        ],
        [
          ["0", "1", "0"],
          ["1", "1", "1"],
        ],
        [
          ["1", "1"],
          ["1", "1"],
        ],
        [["1", "1", "1", "1"]],
        [
          ["0", "1", "1"],
          ["1", "1", "0"],
        ],
        [
          ["1", "1", "0"],
          ["0", "1", "1"],
        ],
      ];

      if (gameStart) {
        for (let i = 0; i < 4; i++) {
          let num = Math.floor(Math.random() * 7);
          let colorIdx = Math.floor(Math.random() * 7);
          this.nextTetrisBlock.push({
            blocks: tetrominos[num],
            color: colors[colorIdx],
            type: num,
            blocksForNext: tetrominosForNext[num],
          });
        }
      } else {
        let num = Math.floor(Math.random() * 7);
        let colorIdx = Math.floor(Math.random() * 7);
        this.nextTetrisBlock.push({
          blocks: tetrominos[num],
          color: colors[colorIdx],
          type: num,
          blocksForNext: tetrominosForNext[num],
        });
      }

      let startIdx = 2;
      this.tetrominoType = this.nextTetrisBlock[0].type;

      for (let i = 0; i < this.nextTetrisBlock[0].blocks.length; i++) {
        for (let j = 0; j < this.nextTetrisBlock[0].blocks[i].length; j++) {
          if (
            this.nextTetrisBlock[0].blocks[i][j] === "1" &&
            this.gameBoard[i][startIdx + j].value === "1"
          ) {
            this.nextTetrisBlock.shift();
            this.gameOver = true;
            return false;
          }
        }
      }

      for (let i = 0; i < this.nextTetrisBlock[0].blocks.length; i++) {
        for (let j = 0; j < this.nextTetrisBlock[0].blocks[i].length; j++) {
          if (this.nextTetrisBlock[0].blocks[i][j] === "1") {
            this.gameBoard[i][startIdx + j].value = "1";
            this.gameBoard[i][startIdx + j].cellActiveStatus = true;
            this.gameBoard[i][startIdx + j].canFade = false;
            this.gameBoard[i][startIdx + j].color =
              this.nextTetrisBlock[0].color;
          }
        }
      }
      this.nextTetrisBlock.shift();
      return true;
    },

    generategameBoard() {
      let id = 0;

      for (let i = 0; i < ROWS; i++) {
        this.gameBoard.push([]);
        for (let j = 0; j < COLS; j++) {
          let cell = {
            value: "0",
            id: id,
            dropTry: false,
            cellActiveStatus: false,
          };
          this.gameBoard[i][j] = cell;
          id++;
        }
      }
    },
  },

  created() {
    this.generategameBoard();
    this.generateTetromino(true);
    this.startGame();
  },
};
</script>

<style>
body {
  display: flex;
  justify-content: center;
  background-color: rgb(234, 166, 98);
}

#app {
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  font-size: medium;
  color: #010506;
  margin-top: 60px;
  display: flex;
  background-color: #9ad9d4;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -o-user-select: none;
  user-select: none;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

@media screen and (max-width: 545px) {
  body {
    -webkit-transform: scale(0.67);
    -moz-transform: scale(0.67);
    transform: scale(0.67);
  }
}

@media screen and (max-width: 373px) {
  body {
    -webkit-transform: scale(0.49);
    -moz-transform: scale(0.49);
    transform: scale(0.49);
    height: 100%;
  }
}
</style>
