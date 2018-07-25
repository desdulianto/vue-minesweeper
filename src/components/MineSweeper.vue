<template>
  <div>
    <p>Minesweeper</p>
    <div v-if="!end">
    <table v-if="loaded">
      <tr v-for="(row, i) in board" :key="i">
        <td v-for="col in row"
            :class="{opened: col.opened && !col.mine, bomb: col.opened && col.mine, marked: col.marked }"
            :key="col.row + ',' + col.col" @click="openBox(col)" @contextmenu.prevent="mark(col)">
            <span v-if="col.opened">{{col.bombs > 0 ? col.bombs : ''}}</span>
        </td>
      </tr>
    </table>
    </div>
    <div style="margin: 0 auto;" v-else>
      <p v-if="status !== ''">you {{status}}!</p>
      <p><label>Size:</label> <input type="number" v-model.number="size"></p>
      <p><label>Bombs:</label> <input type="number" v-model.number="bombs"></p>
      <p><button @click="start">Start</button></p>
    </div>
  </div>
</template>

<script>
function getRandomInt (max) {
  return Math.floor(Math.random() * Math.floor(max))
}

export default {
  name: 'MineSweeper',
  data () {
    return {
      board: [],
      size: 20,
      bombs: 60,
      loaded: false,
      opened: [],
      end: true,
      status: ''
    }
  },
  methods: {
    _getSurroundingBox (row, col) {
      var pos = []
      // top left
      var y = row - 1
      var x = col - 1
      if (y >= 0 && x >= 0) {
        pos.push(this.board[y][x])
      }
      // top
      y = row - 1
      x = col
      if (y >= 0) {
        pos.push(this.board[y][x])
      }
      // top right
      y = row - 1
      x = col + 1
      if (y >= 0 && x < this.size) {
        pos.push(this.board[y][x])
      }
      // right
      y = row
      x = col + 1
      if (x < this.size) {
        pos.push(this.board[y][x])
      }
      // bottom right
      y = row + 1
      x = col + 1
      if (y < this.size && x < this.size) {
        pos.push(this.board[y][x])
      }
      // bottom
      y = row + 1
      x = col
      if (y < this.size) {
        pos.push(this.board[y][x])
      }
      // bottom left
      y = row + 1
      x = col - 1
      if (y < this.size && x >= 0) {
        pos.push(this.board[y][x])
      }
      // left
      y = row
      x = col - 1
      if (x >= 0) {
        pos.push(this.board[y][x])
      }

      return pos
    },
    _countBombs (row, col) {
      var pos = this._getSurroundingBox(row, col)
      return pos.map(x => x.mine ? 1 : 0).reduce((x, y) => x + y)
    },
    _revealBox (row, col) {
      this.board[row][col].opened = true

      if (!this.board[row][col].mine) {
        var pos = this._getSurroundingBox(row, col)

        pos = pos.filter(x => !x.opened && !x.marked)

        var next = []
        for (var i = 0; i < pos.length; i++) {
          var r = pos[i].row
          var c = pos[i].col
          if (!this.board[r][c].mine) {
            this.board[r][c].opened = true
            if (this.board[r][c].bombs === 0) {
              next.push(this.board[r][c])
            }
          }
        }

        // next itiration
        for (i = 0; i < next.length; i++) {
          this._revealBox(next[i].row, next[i].col)
        }
      }
    },
    setup () {
      // setup board
      this.board = []
      for (var i = 0; i < this.size; i++) {
        var row = []
        for (var j = 0; j < this.size; j++) {
          var item = {
            row: i,
            col: j,
            mine: false,
            opened: false,
            marked: false,
            bombs: 0
          }
          row.push(item)
        }
        this.board.push(row)
      }

      // place bomb
      for (i = 0; i < this.bombs;) {
        let x = getRandomInt(this.size)
        let y = getRandomInt(this.size)
        if (!this.board[y][x].mine) {
          this.board[y][x].mine = true
          i++
        }
      }

      // set number in each box
      for (i = 0; i < this.size; i++) {
        for (j = 0; j < this.size; j++) {
          if (this.board[i][j].mine) {
            continue
          }

          this.board[i][j].bombs = this._countBombs(i, j)
        }
      }
      this.loaded = true
    },
    openBox (box) {
      let pos = this.board[box.row][box.col]
      if (!this.end && !pos.marked) {
        if (pos.mine) {
          // lost
          pos.opened = true
          this.end = true
          this.status = 'lost'
        } else {
          if (!pos.opened) {
            this._revealBox(box.row, box.col)
          }
        }

        // check win condition
        this._checkWin()
      }
    },
    mark (box) {
      if (!this.board[box.row][box.col].opened) {
        this.board[box.row][box.col].marked = !this.board[box.row][box.col].marked
        this._checkWin()
      }
    },
    start () {
      this.setup()
      this.end = false
      this.status = ''
    },
    _checkOpenedAll () {
      return this.board.map(x => x.map(y => y.opened || y.marked)
        .reduce((a, b) => a && b)).reduce((a1, b1) => a1 && b1)
    },
    _checkWin () {
      if (!this.end) {
        this.end = this._checkOpenedAll()
        this.status = this.end ? 'win' : ''
      }
    }
  },
  mounted () {
    this.setup()
  }
}
</script>

<style>
table { border: 1px solid black; margin: 0 auto; }
td { border: 1px solid black; width: 2em; height: 2em; }
td { background-color: #828282; }
td.opened { background-color: white; }
td.bomb { background-color: red; }
td.marked { background-color: yellow; }
</style>
