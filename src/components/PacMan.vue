<script>
import 'bootstrap/dist/css/bootstrap.min.css'
import 'bootstrap/dist/js/bootstrap.min.js'
export default {
  name: 'PacMan',
  data () {
    return {
      grid: [],
      gridXMax: 21,
      gridXMin: 0,
      gridYMax: 21,
      gridYMin: 0,
      player: null,
      playerIcon: null,
      mystrawberry: null,
      currentGridSize: {
        x:21,
        y:21,
      },
      ICONS: ['🐶', '🐱', '🐭', '🐨', '🐵', '🐸'],
      LIKESTRAWBERRY:['🍓', '🍀', '🍒', '🍗'],
      CELL_TYPES: {
        block: 'block',
        player: 'player',
        strawberry: 'strawberry',
        finish: 'finish',
      },
      POINTS_FOR_FINISH: 10,
      POINTS_FOR_STRAWBERRY: 2,
      PLAYER_START: { x: 1, y: 1 },
      GRID_SIZE_OPTIONS: {
        '11x11': {
          x: 11,
          y: 11,
        },
        '15x15': {
          x: 15,
          y: 15,
        },
        '21x21': {
          x: 21,
          y: 21,
        },
      },
    };
  },
  methods:{
    maze(inputX, inputY) {
      const x = (inputX - 1) / 2;
      const y = (inputY - 1) / 2;

      let j;
      let n = x * y - 1;

      if (n < 0) {
        // eslint-disable-next-line no-alert
        alert('illegal maze dimensions');

        return;
      }

      const horiz = [];

      for (j = 0; j < x + 1; j += 1) {
        horiz[j] = [];
      }

      const verti = [];

      for (j = 0; j < x + 1; j += 1) {
        verti[j] = [];
      }

      let here = [Math.floor(Math.random() * x), Math.floor(Math.random() * y)];
      const path = [here];
      const unvisited = [];

      for (j = 0; j < x + 2; j += 1) {
        unvisited[j] = [];

        for (let k = 0; k < y + 1; k += 1) {
          unvisited[j]
              .push(j > 0 && j < x + 1 && k > 0 && (j !== here[0] + 1 || k !== here[1] + 1));
        }
      }

      while (n > 0) {
        const potential = [
          [here[0] + 1, here[1]], [here[0], here[1] + 1],
          [here[0] - 1, here[1]], [here[0], here[1] - 1],
        ];

        const neighbors = [];

        for (j = 0; j < 4; j += 1) {
          if (unvisited[potential[j][0] + 1][potential[j][1] + 1]) {
            neighbors.push(potential[j]);
          }
        }

        if (neighbors.length) {
          n -= 1;

          const next = neighbors[Math.floor(Math.random() * neighbors.length)];

          unvisited[next[0] + 1][next[1] + 1] = false;

          if (next[0] === here[0]) {
            horiz[next[0]][(next[1] + here[1] - 1) / 2] = true;
          } else {
            verti[(next[0] + here[0] - 1) / 2][next[1]] = true;
          }

          path.push(here = next);
        } else {
          here = path.pop();
        }
      }

      // eslint-disable-next-line consistent-return
      return {
        x, y, horiz, verti,
      };
    },
    display(m) {
      const mazeArea = [];

      for (let y = 0; y < m.x * 2 + 1; y += 1) {
        const row = [];

        if (y % 2 === 0) {
          for (let x = 0; x < m.y * 2 + 1; x += 1) {
            if (x % 2 === 0) {
              row.push({ type: 'block', x, y });
            } else if (y > 0 && m.verti[y / 2 - 1][Math.floor(x / 2)]) {
              row.push({ type: null, x, y });
            } else {
              row.push({ type: 'block', x, y });
            }
          }
        } else {
          for (let x = 0; x < m.y * 2 + 1; x += 1) {
            if (x % 2 === 0) {
              if (x > 0 && m.horiz[(y - 1) / 2][x / 2 - 1]) {
                row.push({ type: null, x, y });
              } else {
                row.push({ type: 'block', x, y });
              }
            } else {
              row.push({ type: null, x, y });
            }
          }
        }

        // start
        // if (y === 0) {
        //   row[1] = '1';
        // }

        // end
        if (m.x * 2 - 1 === y) {
          row[2 * m.y] = { type: 'finish', x: 2 * m.y, y };
        }

        mazeArea.push(row);
      }

      return mazeArea.slice().reverse();
    },
    move(direction) {
      let newPlayerX = this.player.x;
      let newPlayerY = this.player.y;

      if (direction === 'up' && this.player.y < this.gridYMax) {
        newPlayerY += 1;
      }

      if (direction === 'down' && this.player.y > this.gridYMin) {
        newPlayerY -= 1;
      }

      if (direction === 'left' && this.player.x > this.gridXMin - 1) {
        newPlayerX -= 1;
      }

      if (direction === 'right' && this.player.x < this.gridXMax - 1) {
        newPlayerX += 1;
      }

      if (this.isBlock(newPlayerX, newPlayerY)) {
        return;
      }

      this.player.x = newPlayerX;
      this.player.y = newPlayerY;
    },
    renderGrid() {
      const grid = this.display(this.maze(this.gridXMax, this.gridYMax));

      // init strawberries
      const strawberries = this.getStrawberries(grid, this.gridXMax);
      grid
          .flat()
          .forEach((cell) => {
            strawberries.forEach((strawberry) => {
              if (cell.x === strawberry.x && cell.y === strawberry.y) {
                cell.type = this.CELL_TYPES.strawberry;
              }
            });
          });
      this.grid = grid;
      console.log(this.grid)
    },
    getStrawberries(grid, XMax) {
      const flatted = grid
          .flat()
          .filter((cell) => {
            if ([
              this.CELL_TYPES.block,
              this.CELL_TYPES.player,
              this.CELL_TYPES.finish,
            ].includes(cell.type)) {
              return false;
            }

            if (cell.x === this.PLAYER_START.x && cell.y === this.PLAYER_START.y) {
              return false;
            }

            return true;
          });

      let strawberriesCount = 0;

      switch (XMax) {
        case 11:
          strawberriesCount = 3;
          break;
        case 15:
          strawberriesCount = 5;
          break;
        case 21:
          strawberriesCount = 7;
          break;
        default:
      }

      const items = [];

      for (let i = 0; i < strawberriesCount; i += 1) {
        items.push(flatted.splice(Math.floor(Math.random() * flatted.length), 1)[0]);
      }

      return items;
    },
    renderPlayer() {
      this.grid
          .flat()
          .filter((cell) => cell.type !== this.CELL_TYPES.block)
          .forEach((cell) => {
            if (cell.x === this.player.x && cell.y === this.player.y) {
              cell.type = this.CELL_TYPES.player;
            } else if (cell.type === this.CELL_TYPES.player) {
              cell.type = null;
            }
          });
    },
    beforeMoveHook() {
      this.grid
          .flat()
          .filter((cell) => cell.type !== this.CELL_TYPES.block)
          .filter((cell) => cell.x === this.player.x && cell.y === this.player.y)
          .forEach((cell) => {
            switch (cell.type) {
              case this.CELL_TYPES.finish:
                this.updateGridSize();
                break;
              case this.CELL_TYPES.strawberry:
                break;
              default:
            }
          });
    },
    isBlock(x, y) {
      return this.grid
          .flat()
          .some((cell) => cell.x === x && cell.y === y && cell.type === this.CELL_TYPES.block);
    },
    updateGridSize(payload) {
      if (payload) {
        this.currentGridSize = this.GRID_SIZE_OPTIONS[payload];
      }
      this.gridXMax = this.currentGridSize.x;
      this.gridYMax = this.currentGridSize.y;
      this.renderGrid();
      this.resetPlayer();
    },
    resetPlayer() {
      this.player = { ...this.PLAYER_START };
      this.renderPlayer();
    },
    randomPlayerIcon() {
      return [...this.ICONS].sort(() => 0.5 - Math.random())[0];
    },
    randomStrawberry() {
        return [...this.LIKESTRAWBERRY].sort(() => 0.5 - Math.random())[0];
    },
    keyboardHandler(event) {
      const arrows = (code) => {
        switch (code) {
          case 'ArrowUp':
            this.move('up');
            break;
          case 'ArrowDown':
            this.move('down');
            break;
          case 'ArrowLeft':
            this.move('left');
            break;
          case 'ArrowRight':
            this.move('right');
            break;
          default:
        }
      };

      switch (event.code) {
        case 'Space':
          break;
        default:
          arrows(event.code);
      }
    },
  },
  watch: {
    player: {
      handler() {
        this.beforeMoveHook();
        this.renderPlayer();
      },
      deep: true,
    },
  },
  mounted() {
    this.player = { ...this.PLAYER_START };
    this.playerIcon = this.randomPlayerIcon();
    this.mystrawberry = this.randomStrawberry();
  },
  created() {
    document.addEventListener('keydown', (event) => this.keyboardHandler(event));
    this.gridXMax = this.currentGridSize.x;
    this.gridYMax = this.currentGridSize.y;
    //console.log(this.gridXMax)
    this.renderGrid();
    this.resetPlayer();
  },
}
</script>

<template>
  <div class="d-flex">
    <div class="main-container">
      <div class="grid-wrapper">
        <div class="d-flex" v-for="row in grid">
          <div
              class="cell"
              :class="{
              'cell-block': cell.type === CELL_TYPES.block,
              'cell-player': cell.type === CELL_TYPES.player
            }"
              v-for="cell in row"
          >
            <div v-if="cell.type === CELL_TYPES.block" class="cell-icon icon-brick"></div>
            <div v-if="cell.type === CELL_TYPES.player" class="cell-emoji">{{ playerIcon }}</div>
            <div v-if="cell.type === CELL_TYPES.strawberry" class="cell-emoji">{{ mystrawberry }}</div>
            <div v-if="cell.type === CELL_TYPES.finish" class="cell-emoji">🏁</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.grid-wrapper {
  width: fit-content;
  border: 1px solid black;
  background-image: url(../assets/stone_2.jpg);
  background-size: 80px;
  position: relative;
}
.cell {
  width: 40px;
  height: 40px;
}
.cell-icon.icon-brick {
  box-shadow: 0 0 0 1px rgba(42, 41, 37, 0.7) inset;
  background-image: url(../assets/stone.jpg);
  white-space:nowrap;
}
.cell-block {
    box-shadow: 0 0 0 1px rgba(42, 41, 37, 0.7) inset;
    background-image: url(../assets/stone.jpg);
    white-space:nowrap;
}


.cell-player {
   background-color: rgba(78, 239, 21, 0.2);
   white-space:nowrap;
}

.cell-icon {
  height: 100%;
  background-size: contain;
  background-repeat: no-repeat;
  white-space:nowrap;
}

.cell-emoji {
  padding: 4px;
  font-size: 22px;
  white-space:nowrap;
}
.custom-modals {
  align-items: center;
  display: flex;
  flex-direction: column;
  height: 100%;
  justify-content: center;
  letter-spacing: 2px;
  opacity: 0.7;
  position: absolute;
  width: 100%;
  z-index: 1;
}

</style>
