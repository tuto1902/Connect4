<template>
  <div id="app">
    <h1 v-show="!gameOver">
      Player <span v-if="turn == player1">1</span><span v-else>2</span>
    </h1>
    <h1 v-show="gameOver">
      Player <span v-if="turn == player1">1</span><span v-else>2</span> Wins!
    </h1>
    <div class="settings">
      <div>
        <input type="radio" id="easy" v-model="difficulty" value="easy" />
        <label for="easy">Easy</label>
        <input type="radio" id="hard" v-model="difficulty" value="hard" />
        <label for="easy">Hard</label>
      </div>
      <div>
        <input
          type="radio"
          id="single-p"
          @click="resetBoard()"
          v-model="mode"
          value="SinglePlayer"
        />
        <label for="single-p">Versus AI</label>
        <input
          type="radio"
          id="two-p"
          @click="resetBoard()"
          v-model="mode"
          value="TwoPlayers"
        />
        <label for="two-p">Two Player Mode</label>
      </div>
    </div>
    <div class="wrapper">
      <div class="board">
        <div
          class="column"
          v-for="(column, columnIndex) in board[0]"
          :key="'col-' + columnIndex"
          :style="{ left: columnIndex * 50 + 'px' }"
          @click="takeTurn(columnIndex)"
        ></div>
        <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
          <svg
            v-for="(column, columnIndex) in row"
            :key="columnIndex"
            width="50"
            height="50"
          >
            <circle
              cx="25"
              cy="25"
              r="20"
              :id="'circle-' + rowIndex + '-' + columnIndex"
              :class="{
                empty: column == empty,
                red: column == red,
                yellow: column == yellow
              }"
            />
          </svg>
        </div>
      </div>
    </div>
    <button v-show="gameOver" type="button" @click="resetBoard()">Reset</button>
  </div>
</template>

<script>
import * as connect4 from "@/connect4.js";

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function highlightPieces(pieces) {
  pieces.forEach(piece => {
    document
      .getElementById("circle-" + piece.row + "-" + piece.col)
      .classList.add("flash");
  });
}

export default {
  name: "App",

  data() {
    return {
      board: [],
      turn: 0,
      player1: 0,
      player2: 1,
      red: connect4.Red,
      yellow: connect4.Yellow,
      empty: connect4.Empty,
      gameOver: false,
      difficulty: "easy",
      mode: "SinglePlayer"
    };
  },

  methods: {
    takeTurn(column) {
      if (
        !this.gameOver &&
        (this.turn == this.player1 ||
          (this.turn == this.player2 && this.mode == "TwoPlayers")) &&
        connect4.isValidColumn(this.board, column)
      ) {
        let row = connect4.getOpenRow(this.board, column);
        let color = this.turn == this.player1 ? this.red : this.yellow;
        connect4.dropPiece(this.board, row, column, color);
        if (connect4.isWinningMove(this.board, color)) {
          this.gameOver = true;
        } else {
          this.turn += 1;
          this.turn = this.turn % 2;
          if (this.mode == "SinglePlayer") {
            sleep(1000).then(() => {
              this.AITurn();
            });
          }
        }
        console.log(this.board);
      }
    },
    AITurn() {
      let column = 0;
      if (this.difficulty == "easy") {
        column = this.selectBestColumn();
      } else {
        let result = connect4.minimax(this.board, 2, -Infinity, Infinity, true);
        column = result.column;
      }
      let row = connect4.getOpenRow(this.board, column);
      connect4.dropPiece(this.board, row, column, this.yellow);
      if (connect4.isWinningMove(this.board, this.yellow)) {
        this.gameOver = true;
      } else {
        this.turn += 1;
        this.turn = this.turn % 2;
      }
    },
    selectBestColumn() {
      let validColumns = connect4.getValidColumns(this.board);
      let highestScore = -1000;
      let column = Math.floor(Math.random() * validColumns.length);
      for (let i = 0; i < validColumns.length; i++) {
        let newColumn = validColumns[i];
        let row = connect4.getOpenRow(this.board, newColumn);
        let boardCopy = connect4.copyBoard(this.board);
        connect4.dropPiece(boardCopy, row, newColumn, this.yellow);
        let newScore = connect4.boardScore(boardCopy);
        if (newScore > highestScore) {
          highestScore = newScore;
          column = newColumn;
        }
      }
      return column;
    },
    resetBoard() {
      this.board = connect4.createBoard();
      this.gameOver = false;
      this.turn = this.player1;
    }
  },

  created() {
    this.board = connect4.createBoard();
    // console.log(this.board);
  },

  updated() {
    if (this.gameOver) {
      let color = this.turn == this.player1 ? this.red : this.yellow;
      let winningPieces = connect4.getWinningPieces(this.board, color);
      highlightPieces(winningPieces);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.wrapper {
  display: flex;
  justify-content: center;
}

.board {
  position: relative;
}

.row {
  display: flex;
  background-color: #0f55ff;
}

.column {
  box-sizing: border-box;
  position: absolute;
  top: 0;
  width: 50px;
  height: 100%;
  transition: background-color 0.1s ease-in-out;
  background-color: transparent;
}

.column:hover {
  cursor: pointer;
  background-color: rgba(255, 255, 255, 0.2);
}

circle.empty {
  fill: #fff;
}

circle.red {
  fill: #d50000;
}

circle.yellow {
  fill: #dad400;
}

circle.red.flash {
  animation: pulse-red 1.2s ease-in-out infinite;
}

circle.yellow.flash {
  animation: pulse-yellow 1.2s ease-in-out infinite;
}

@keyframes pulse-yellow {
  0% {
    fill: #dad400;
  }

  50% {
    fill: #fffdab;
  }

  100% {
    fill: #dad400;
  }
}

@keyframes pulse-red {
  0% {
    fill: #d50000;
  }

  50% {
    fill: #ff8f8f;
  }

  100% {
    fill: #d50000;
  }
}

.settings {
  display: flex;
  flex-direction: column;
  width: 50%;
  margin: auto;
}

.settings div {
  padding: 10px;
}

.settings label {
  margin-right: 10px;
}
</style>
