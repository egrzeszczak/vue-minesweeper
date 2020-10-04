<template>
  <div class="minesweeper" @contextmenu.prevent="">
    <transition name="fade">
        <div class="minefield" :style="minefieldStyle" v-if="minefield.length > 0" 
            :class="{
                minefield_green: (state === 1),
                minefield_red: (state === -1),
            }">
        <button
            v-for="field in minefield"
            :key="field.id"
            :class="{
            flag: (field.flag && !field.show),
            show: field.show,
            bomb: (field.show && field.bomb === 1),
            }"
            :style="{
                color: field.nearbyColor,    
            }"
            @click="fieldLeftClick(field.id)"
            @contextmenu.prevent="fieldRightClick(field.id)"
        >
            <span v-if="field.show && !(field.bomb === 1)">{{ field.nearby }}</span>
            <span v-if="field.flag && !field.show">&#128681;</span>
            <span v-if="field.show && field.bomb === 1">&#128163;</span>
        </button>
        </div>
    </transition>

    <transition name="fade">
        <div class="menu" v-if="minefield.length == 0">
            <h1>Saper</h1>
            <button @click="setDifficulty('easy')">Easy</button>
            <button @click="setDifficulty('medium')">Medium</button>
            <button @click="setDifficulty('hard')">Hard</button>
            <button @click="setDifficulty('very hard')">Very hard</button>
        </div>
        <div class="back-menu" v-if="minefield.length > 0 && state == 0">
            <button @click="gameOver(-1)">Surrender</button>
        </div>
    </transition>
  </div>
</template>


<script>
// &#128163; - BOMB
// &#128681; - FLAG
export default {
  data() {
    return {
      state: 0, // 0 - playing, 1 - won, -1 - lost  
      minefield: [],
      width: 0,
      height: 0,
      bombCount: 0,
      flagsPlanted: 0,
    };
  },
  methods: {
    setDifficulty(difficulty){
        this.minefield = [];
        switch (difficulty) {
            case "easy":
                setTimeout(() => {
                    this.minefield = this.newMinefield(8, 8);
                    }, 250);
                break;
            case "medium":
                setTimeout(() => {
                    this.minefield = this.newMinefield(16, 12);
                    }, 250);
                break;
            case "hard":
                setTimeout(() => {
                    this.minefield = this.newMinefield(20, 15);
                    }, 250);
                break;
            case "very hard":
                setTimeout(() => {
                    this.minefield = this.newMinefield(30, 20);
                    }, 250);
                break;
            default:
                break;
        }
    },
    newMinefield(w, h) {
        let res = [];
        this.state = 0;
        this.width = w;
        this.height = h;
        this.bombCount = 0;
        this.flagsPlanted = 0;
      console.group("New Game", new Date());
      console.info(
        "Creating new minefield ( W:",
        this.width,
        "H:",
        this.height,
        ")"
      );
      for (let y = 0; y < w; y++) {
        for (let x = 0; x < h; x++) {
          let armWithBomb = Math.random() >= 0.9 ? 1 : 0;
          this.bombCount += armWithBomb;
          res.push({
            id: h * y + x,
            pos_x: x,
            pos_y: y,
            bomb: armWithBomb,
            show: false,
            flag: false,
            nearby: 0,
            nearbyColor: "black",
          });
        }
      }
      console.info(
        "Minefield created \n",
        "- Bombs:",
        this.bombCount,
        "/",
        this.width * this.height,
        "\n",
        "- Bombs(%):",
        Math.round((this.bombCount / (this.width * this.height)) * 100),
        "%"
      );
      console.groupEnd();

      return res;
    },
    fieldLeftClick(index) {
      if (this.minefield[index].bomb === 1) {
        this.gameOver(index);
      } else {
        if (!this.minefield[index].show) {
          // Click lock
          this.minefield[index].show = true;
          let nearby = 0;

          // LU_CORNER CHECK --- OK
          if (index == 0) {
            nearby =
              this.minefield[1].bomb +
              this.minefield[this.width].bomb +
              this.minefield[this.width + 1].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(1);
              this.fieldLeftClick(this.width);
              this.fieldLeftClick(this.width + 1);
            }
          }
          // RU_CORNER CHECK --- OK
          else if (index == this.width - 1) {
            nearby =
              this.minefield[this.width - 2].bomb +
              this.minefield[this.width * 2 - 1].bomb +
              this.minefield[this.width * 2 - 2].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(this.width - 2);
              this.fieldLeftClick(this.width * 2 - 1);
              this.fieldLeftClick(this.width * 2 - 2);
            }
          }
          // LD_CORNER CHECK --- OK
          else if (index == this.height * this.width - this.width) {
            nearby =
              this.minefield[this.height * this.width - this.width + 1].bomb +
              this.minefield[this.height * this.width - this.width * 2].bomb +
              this.minefield[this.height * this.width - this.width * 2 + 1]
                .bomb;
            if (nearby == 0) {
              this.fieldLeftClick(this.height * this.width - this.width + 1);
              this.fieldLeftClick(this.height * this.width - this.width * 2);
              this.fieldLeftClick(this.height * this.width - this.width * 2 + 1);
            }
          }
          // RD_CORNER CHECK
          else if (index == this.height * this.width - 1) {
            nearby =
              this.minefield[this.height * this.width - 2].bomb +
              this.minefield[this.height * this.width - this.width - 1].bomb +
              this.minefield[this.height * this.width - this.width - 2].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(this.height * this.width - 2);
              this.fieldLeftClick(this.height * this.width - this.width - 1);
              this.fieldLeftClick(this.height * this.width - this.width - 2);
            }
          }
          // TOP CHECK --- OK
          else if (index > 0 && index < this.width) {
            nearby =
              this.minefield[index - 1].bomb +
              this.minefield[index + 1].bomb +
              this.minefield[index + this.width - 1].bomb +
              this.minefield[index + this.width].bomb +
              this.minefield[index + this.width + 1].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(index - 1);
              this.fieldLeftClick(index + 1);
              this.fieldLeftClick(index + this.width - 1);
              this.fieldLeftClick(index + this.width);
              this.fieldLeftClick(index + this.width + 1);
            }
          }
          // LEFT CHECK --- OK
          else if (
            index > 0 &&
            index < this.height * this.width - this.width &&
            index % this.width == 0
          ) {
            nearby =
              this.minefield[index - this.width].bomb +
              this.minefield[index - this.width + 1].bomb +
              this.minefield[index + 1].bomb +
              this.minefield[index + this.width].bomb +
              this.minefield[index + this.width + 1].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(index - this.width);
              this.fieldLeftClick(index - this.width + 1);
              this.fieldLeftClick(index + 1);
              this.fieldLeftClick(index + this.width);
              this.fieldLeftClick(index + this.width + 1);
            }
          }
          // RIGHT CHECK --- OK
          else if (
            index % this.width == this.width - 1 &&
            index != this.width - 1
          ) {
            nearby =
              this.minefield[index - this.width - 1].bomb +
              this.minefield[index - this.width].bomb +
              this.minefield[index - 1].bomb +
              this.minefield[index + this.width - 1].bomb +
              this.minefield[index + this.width].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(index - this.width - 1);
              this.fieldLeftClick(index - this.width);
              this.fieldLeftClick(index - 1);
              this.fieldLeftClick(index + this.width - 1);
              this.fieldLeftClick(index + this.width);
            }
          }
          // DOWN CHECK --- OK
          else if (
            index > this.height * this.width - this.width &&
            index < this.height * this.width - 1
          ) {
            nearby =
              this.minefield[index - 1].bomb +
              this.minefield[index + 1].bomb +
              this.minefield[index - this.width - 1].bomb +
              this.minefield[index - this.width].bomb +
              this.minefield[index - this.width + 1].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(index - 1);
              this.fieldLeftClick(index + 1);
              this.fieldLeftClick(index - this.width - 1);
              this.fieldLeftClick(index - this.width);
              this.fieldLeftClick(index - this.width + 1);
            }
          }
          // CENTER CHECK --- OK
          else {
            nearby =
              this.minefield[index - this.width - 1].bomb +
              this.minefield[index - this.width].bomb +
              this.minefield[index - this.width + 1].bomb +
              this.minefield[index - 1].bomb +
              this.minefield[index + 1].bomb +
              this.minefield[index + this.width - 1].bomb +
              this.minefield[index + this.width].bomb +
              this.minefield[index + this.width + 1].bomb;
            if (nearby == 0) {
              this.fieldLeftClick(index - this.width - 1);
              this.fieldLeftClick(index - this.width);
              this.fieldLeftClick(index - this.width + 1);
              this.fieldLeftClick(index - 1);
              this.fieldLeftClick(index + 1);
              this.fieldLeftClick(index + this.width - 1);
              this.fieldLeftClick(index + this.width);
              this.fieldLeftClick(index + this.width + 1);
            }
          }
          this.minefield[index].nearby = nearby;
          switch(nearby){
            case 0: { this.minefield[index].nearbyColor = "rgb(120, 120, 120)"; break;}
            case 1: { this.minefield[index].nearbyColor = "rgb(0,0,255)"; break; }
            case 2: { this.minefield[index].nearbyColor = "rgb(0, 102,0)"; break; }
            case 3: { this.minefield[index].nearbyColor = "rgb(255, 0, 0)"; break; }
            case 4: { this.minefield[index].nearbyColor = "rgb(0, 0, 102)"; break; }
            case 5: { this.minefield[index].nearbyColor = "rgb(102, 0, 0)"; break; }
            case 6: { this.minefield[index].nearbyColor = "rgb(125,104, 0)"; break; }
            case 7: { this.minefield[index].nearbyColor = "rgb(200, 102, 0)"; break; }
            case 8: { this.minefield[index].nearbyColor = "rgb(255, 0, 255)"; break; }
          }
        }
      }
    },
    fieldRightClick(index) {
      if (this.minefield[index].flag) {
        this.flagsPlanted--;
      } else this.flagsPlanted++;
      this.minefield[index].flag = !this.minefield[index].flag;
      if (this.bombCount == this.flagsPlanted) {
        this.gameSuccess();
      }
    },
    gameOver(index){
        console.error("You lost");
        this.minefield.forEach((field) => {
            setTimeout(() => {
                if(field.bomb === 1) { field.show = true; }
            }, 150);
        });
        if(index > 0) this.minefield[index].show = true;
        this.state = -1;
        setTimeout(() => {
            this.minefield = [];
        }, 3000);
    },
    gameSuccess(){
        this.minefield.forEach((field) => {
            if(!(field.bomb === 1)) this.fieldLeftClick(field.id)
        })
        console.info("You win.");
        this.state = 1;
    }
  },
  computed: {
    minefieldStyle() {
      return {
        gridTemplateColumns: `repeat(${this.width}, minmax(24px, 27px))`,
      };
    },
  },
};
</script>

<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.minesweeper {
  min-height: 100vh;
  background: white;
  text-align: center;
  font-family: "Lucida Console", Monaco, monospace;
  h1{
      letter-spacing: 2px;
  }
  color: #2c3e50;
  display: grid;
  align-items: center;
  justify-items: center;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  .minefield {
    padding: 0.25rem;
    display: grid;
    background: white;
    box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2), 0 2px 10px 0 rgba(0, 0, 0, 0.19);
    button {
      background: #232730;
      border-radius: 3px;
      border: 1px solid transparent;
      padding: 0;
      width: 24px;
      height: 24px;
      margin: 1px 2px;
      font-weight: bold;
      transition: .45s all cubic-bezier(0.175, 0.885, 0.32, 1.275);
      &:hover {
        background: #536891;
        border: 1px solid rgb(204, 204, 204);
        cursor: pointer;
      }
    }
    .show {
      background: white;
      color: black;
      box-shadow: none;

      &:hover {
        border: 1px solid transparent;
        background: inherit;
      }
    }
    .flag {
      background: linear-gradient(32deg, #e08d8d, #ff7878);
      border: 1px solid transparent;
      box-shadow: 0 0px 1px 0 #8f2c2c, 0 0px 2px 0 #8f2c2c;
      &:hover {
        border: 1px solid #ff0000;
        background: linear-gradient(32deg, #e68a8a, #ff7c7c);
      }
    }
    .bomb{
        background: red;
    }
  }
  .minefield_green{
    box-shadow: 1px 6px 6px 1px rgba(51, 255, 0, 0.438), 0 1px 5px 0 rgba(72, 255, 0, 0.466);
    .flag {
      background: linear-gradient(32deg, #a0e08d, #9eff78);
      border: 1px solid transparent;
      box-shadow: 0 0px 1px 0 #348f2c, 0 0px 2px 0 #488f2c;
      &:hover {
        border: 1px solid #51ff00;
        background: linear-gradient(32deg, #a7e68a, #96ff7c);
      }
    }
  }
  .minefield_red{
      box-shadow: 1px 6px 6px 1px rgba(255, 0, 0, 0.438), 0 1px 5px 0 rgba(255, 0, 0, 0.466);
  }
}
</style>