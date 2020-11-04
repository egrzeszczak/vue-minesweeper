<template>
	<div class="minesweeper" @contextmenu.prevent="">
		<transition name="fade">
			<div
				class="minefield"
				:style="minefieldStyle"
				v-if="minefield.length > 0"
				:class="{
					minefield_green: state === 1,
					minefield_red: state === -1,
				}"
			>
				<button
					v-for="field in minefield"
					:key="field.id"
					:class="{
						flag: field.flag && !field.show,
						show: field.show,
						bomb: field.show && field.bomb === 1,
					}"
					:style="{
						color: field.nearbyColor,
					}"
					@click="fieldLeftClick(field.id)"
					@contextmenu.prevent="fieldRightClick(field.id)"
				>
					<span v-if="field.show && !(field.bomb === 1)">{{
						field.nearby
					}}</span>
					<span v-if="field.flag && !field.show">&#128681;</span>
					<span v-if="field.show && field.bomb === 1">&#128163;</span>
				</button>
			</div>
		</transition>
		<transition name="fade">
			<div class="buttons">
				<span v-if="minefield.length == 0">Saper</span>
				<span
					v-if="minefield.length > 0"
					:class="{ red: flagsPlanted > mineCount }"
				>
					mines: {{ mineCount }}, flags planted: {{ flagsPlanted }}
				</span>
				<button v-if="minefield.length > 0 && state == 0" @click="goBack">
					&larr;
				</button>

				<div class="start-button" v-if="minefield.length == 0 || state != 0">
					<button
						v-for="diff in difficulties"
						:key="diff.name"
						@click="setDifficulty(diff)"
					>
						{{ diff.name }}
					</button>
				</div>
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
			// Minefield width
			width: 0,
			// Minefield height
			height: 0,
			// State, (0) Game running, (1) Won, (-1) Lost, (2) Default at start
			state: 2,
			// Minefield array of objects
			minefield: [],
			// Count of mines deployed
			mineCount: 0,
			// Current probability
			mineProbability: 0.5, // hard=0........0.5........easy=1
			// Flags planted by player
			flagsPlanted: 0,
			// Minefield lock
			minefieldLock: false,
			// Difficulties
			difficulties: {
				easy: {
					name: "easy",
					p: 0.9,
					w: 8,
					h: 8,
				},
				medium: {
					name: "medium",
					p: 0.8,
					w: 14,
					h: 14,
				},
				hard: {
					name: "hard",
					p: 0.6,
					w: 20,
					h: 15,
				},
				"very hard": {
					name: "very hard",
					p: 0.5,
					w: 40,
					h: 15,
				},
			},
		};
	},
	methods: {
		setDifficulty(difficulty) {
			// Clear minefield
			this.minefield = [];
			// Set new probability
			this.mineProbability = difficulty.p;
			// Create new minefield
			this.minefield = this.newMinefield(difficulty.w, difficulty.h);
		},
		newMinefield(w, h) {
			// New game vars
			let res = [];
			this.state = 0;
			this.width = w;
			this.height = h;
			this.mineCount = 0;
			this.flagsPlanted = 0;
			this.minefieldLock = false;
			// Console log
			console.group("New Game", new Date());
			console.info(
				"Creating new minefield ( W:",
				this.width,
				"H:",
				this.height,
				")"
			);
			// Mine planter
			for (let y = 0; y < w; y++) {
				for (let x = 0; x < h; x++) {
					let armWithBomb = Math.random() >= this.mineProbability ? 1 : 0;
					this.mineCount += armWithBomb;
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
			// Console log
			console.info(
				"Minefield created \n",
				"- Bombs:",
				this.mineCount,
				"/",
				this.width * this.height,
				"\n",
				"- Bombs(%):",
				Math.round((this.mineCount / (this.width * this.height)) * 100),
				"%"
			);
			console.groupEnd();
			return res;
		},
		fieldLeftClick(index) {
			if (this.minefield[index].bomb === 1) {
				// Step on a mine
				this.gameOver(index);
			} else if (this.minefieldLock) {
				// Game ended. Cant click
			} else {
				if (!this.minefield[index].show) {
          // Uncheck before revealing
          if(this.minefield[index].flag){
            this.fieldRightClick(index);
          }
					this.minefield[index].show = true;
					let nearby = 0;

          // It is what it is
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
							this.fieldLeftClick(
								this.height * this.width - this.width * 2 + 1
							);
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
					switch (nearby) {
						case 0: {
							this.minefield[index].nearbyColor = "rgb(120, 120, 120)";
							break;
						}
						case 1: {
							this.minefield[index].nearbyColor = "rgb(0,0,255)";
							break;
						}
						case 2: {
							this.minefield[index].nearbyColor = "rgb(0, 102,0)";
							break;
						}
						case 3: {
							this.minefield[index].nearbyColor = "rgb(255, 0, 0)";
							break;
						}
						case 4: {
							this.minefield[index].nearbyColor = "rgb(0, 0, 102)";
							break;
						}
						case 5: {
							this.minefield[index].nearbyColor = "rgb(102, 0, 0)";
							break;
						}
						case 6: {
							this.minefield[index].nearbyColor = "rgb(125,104, 0)";
							break;
						}
						case 7: {
							this.minefield[index].nearbyColor = "rgb(200, 102, 0)";
							break;
						}
						case 8: {
							this.minefield[index].nearbyColor = "rgb(255, 0, 255)";
							break;
						}
					}
				}
			}
		},
		fieldRightClick(index) {
			if (!this.minefieldLock) {
        
        // ?
        if(!(this.minefield[index].show)){
          if (this.minefield[index].flag) 
            this.flagsPlanted--;
          else
            this.flagsPlanted++;
          this.minefield[index].flag = !this.minefield[index].flag;
        }


				if (this.mineCount == this.flagsPlanted) {
					let minesFound = 0;
					this.minefield.forEach((field) => {
						if (field.flag && field.bomb && !field.show) {
							minesFound++;
							console.log(field.id);
						}
					});
					if (minesFound === this.mineCount) this.gameSuccess();
				}
			}
		},
		gameOver(index) {
			// You lost dont click anything
			this.minefieldLock = true;
			// State (-1) Lost
			this.state = -1;
			// Log
			console.error("You lost");
			// Reveal all exploding bombs
			this.minefield.forEach((field) => {
				setTimeout(() => {
					if (field.bomb === 1) {
						field.show = true;
					}
				}, 150);
			});
			// Code to surrender (index === -1)
			if (index > 0) this.minefield[index].show = true;
		},
		gameSuccess() {
			// You won already. Dont click anything
			this.minefieldLock = true;
			// Reveal all fields
			this.minefield.forEach((field) => {
				if (!(field.bomb === 1)) this.fieldLeftClick(field.id);
			});
			// Write to console
			console.info("You win.");
			// State is (1) Won
			this.state = 1;
		},
		goBack() {
			// &larr;
			this.gameOver(-1);
			this.minefield = [];
		},
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
.red {
	color: red;
}
.fade-enter-active,
.fade-leave-active {
	transition: all 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
	opacity: 0;
}
.buttons {
	position: absolute;
	top: 90vh;
	display: flex;
	align-items: center;
	span,
	button {
		padding: 0.25rem 2rem;
	}
	button {
		border: none;
		background: none;
		padding: 0.25rem 2rem;
		border-left: 1px solid #2c3e50;
		&:hover {
			background: #2c3e50;
			color: white;
			cursor: pointer;
		}
	}
}
.minesweeper {
	min-height: 100vh;
	background: white;
	text-align: center;
	font-family: "Lucida Console", Monaco, monospace;
	h1 {
		letter-spacing: 2px;
	}
	color: #2c3e50;
	display: grid;
	justify-items: center;
	align-items: center;
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
			transition: 0.45s all cubic-bezier(0.175, 0.885, 0.32, 1.275);
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
		.bomb {
			background: red;
		}
	}
	.minefield_green {
		box-shadow: 1px 6px 6px 1px rgba(51, 255, 0, 0.438),
			0 1px 5px 0 rgba(72, 255, 0, 0.466);
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
	.minefield_red {
		box-shadow: 1px 6px 6px 1px rgba(255, 0, 0, 0.438),
			0 1px 5px 0 rgba(255, 0, 0, 0.466);
	}
}
</style>
