<template>
	<h1>Démineur</h1>
	Click sur le petit bonhomme jaune pour jouer
	<div class="mainContainer">
		<div class="headContainer">
			<div class="headContainerItem">
				{{ flags < 100 ? "0" : "" }}{{ flags }}
			</div>
			<div @click="start" class="headContainerItem">
				<p class="icon">{{ iconContent }}</p>
			</div>
			<div class="headContainerItem">
				{{ timerVal < 100 ? "0" : "" }}{{ timerVal < 10 ? "0" : ""
				}}{{ timerVal }}
			</div>
		</div>
		<div id="gridContainer">
			<div class="row" v-for="(row, indexRow) in rows">
				<Cell
					v-for="(col, indexCol) in cols"
					:key="row.toString() + col.toString()"
					:pos="[row, col]"
					:rows="rows"
					:cols="cols"
					@emptyZone="emptyZone"
					@resetEmptyZone="resetEmptyZone"
					@flagUsed="flagUsed"
					@checkWin="checkWin"
					@game-over="gameOver"
					:isStarted="isStarted"
					:show="show"
					:flags="flags"
					:content="gridContent[indexRow][indexCol]"
					:neighbourEmpty="neighbourEmpty"
				/>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import Cell from "./components/Cell.vue";
import { ref, reactive, watch } from "vue";
const iconContent = ref(":)"); // début :) / perdu :( / gagné :D
const nbMines = ref(10);
const timerVal = ref(0);
const rows = ref(9);
const cols = ref(9);
const isStarted = ref(false);
const show = ref(false);
const neighbourEmpty = ref([100, 100]);
// const content = ref(10);
let gridContent = reactive([] as number[][]);

for (let i = 0; i < rows.value; i++) {
	gridContent.push([]);
	for (let j = 0; j < cols.value; j++) {
		gridContent[i].push(0);
	}
}

const start = () => {
	stopTimer();
	// reset values
	gridContent.map(
		(row, rowIndex) => (gridContent[rowIndex] = row.map(() => 0))
	);
	neighbourEmpty.value = [100, 100];
	iconContent.value = ":)";
	isStarted.value = true;
	timerVal.value = 0;
	flags.value = 10;
	/////////////////////////////////////////////////////////////////
	// ça marche (pour reset le show) ... mais pourquoi ?
	// direct show.value = false marchait pas... et sans setTimeout() non plus
	// changement props asynchrone (y'a un delai ?)
	show.value = true;
	setTimeout(() => {
		show.value = false;
	}, 10);
	/////////////////////////////////////////////////////////////////

	randomFillBombs();
	startTimer();
};

// function to display bombs
const randomFillBombs = () => {
	// random cell (row and col), { nbMines } time
	for (let i = 0; i < nbMines.value; i++) {
		const randomRow = Math.floor(Math.random() * rows.value);
		const randomCol = Math.floor(Math.random() * cols.value);
		// if a bomb is already there => random once more
		if (gridContent[randomRow][randomCol] === 10) {
			i--;
		}
		gridContent[randomRow][randomCol] = 10;
	}
	calculate();
};

// function to display numbers near bombs
const calculate = () => {
	gridContent.map((row, indexRow) =>
		row.map((col, indexCol) => {
			if (gridContent[indexRow][indexCol] === 10) {
				// row above, from left to right:
				if (
					indexRow - 1 !== -1 &&
					indexCol - 1 !== -1 &&
					gridContent[indexRow - 1][indexCol - 1] !== 10
				) {
					gridContent[indexRow - 1][indexCol - 1] += 1;
				}
				if (indexRow - 1 !== -1 && gridContent[indexRow - 1][indexCol] !== 10) {
					gridContent[indexRow - 1][indexCol] += 1;
				}
				if (
					indexRow - 1 !== -1 &&
					indexCol + 1 !== cols.value &&
					gridContent[indexRow - 1][indexCol + 1] !== 10
				) {
					gridContent[indexRow - 1][indexCol + 1] += 1;
				}

				// same row, from left to right:
				if (indexCol - 1 !== -1 && gridContent[indexRow][indexCol - 1] !== 10) {
					gridContent[indexRow][indexCol - 1] += 1;
				}
				if (
					indexCol + 1 !== cols.value &&
					gridContent[indexRow][indexCol + 1] !== 10
				) {
					gridContent[indexRow][indexCol + 1] += 1;
				}

				// row below, from left to right:
				if (
					indexRow + 1 !== rows.value &&
					indexCol - 1 !== -1 &&
					gridContent[indexRow + 1][indexCol - 1] !== 10
				) {
					gridContent[indexRow + 1][indexCol - 1] += 1;
				}
				if (
					indexRow + 1 !== rows.value &&
					gridContent[indexRow + 1][indexCol] !== 10
				) {
					gridContent[indexRow + 1][indexCol] += 1;
				}
				if (
					indexRow + 1 !== rows.value &&
					indexCol + 1 !== cols.value &&
					gridContent[indexRow + 1][indexCol + 1] !== 10
				) {
					gridContent[indexRow + 1][indexCol + 1] += 1;
				}
			}
		})
	);
};

/////////////    timer     /////////////////
let timerId: number;
const startTimer = () => {
	timerId = setInterval(() => timerVal.value++, 1000);
};
const stopTimer = () => {
	clearInterval(timerId);
};

////////////   flags   //////////////////////
const flags = ref(nbMines.value);
// emit listened -> if flag added by a child component, nb = -1, else nb = 1
// if flags.value === 0 child component can't add a flag
const flagUsed = (nb: number) => {
	flags.value += nb;
};

// when a child is clicked -> send pos to all children to check if they are empty too, to reveal the empty zone
const emptyZone = (emptyCellClicked: number[]) => {
	// neighbourEmpty is a props passed to children
	neighbourEmpty.value = emptyCellClicked;
};
const resetEmptyZone = () => {
	neighbourEmpty.value = [100, 100];
};

const checkWin = () => {
	// genre on parcours ... je sais pas quoi et si chaque case value !== 10 est show === true => c'est gagné
	const grid = document.getElementById("gridContainer") as HTMLElement;
	let goodCellsDisplayed = 0;
	for (let i = 0; i < grid.children.length; i++) {
		for (let j = 0; j < grid.children[i].children.length; j++) {
			const cell = grid.children[i].children[j].children;

			// class = bomb pour là où y'a des bombs
			const isNotABomb = cell[0].classList[0] !== "bomb";
			// const cellValClass = grid.children[i].children[j].children[0].classList;

			// class displayNone là où le cache est enlevé = (show = true)
			const isDisplayed = cell[1].classList[0] === "displayNone";

			if (isDisplayed && isNotABomb) {
				goodCellsDisplayed++;
			}
		}
	}
	if (goodCellsDisplayed === rows.value * cols.value - nbMines.value) {
		show.value = true;
		iconContent.value = ":D";
		isStarted.value = false;
		stopTimer();
		setTimeout(() => {
			alert("win !!!!!!!!");
		}, 150);
	}
};

const gameOver = () => {
	iconContent.value = ":(";
	show.value = true;
	isStarted.value = false;
	stopTimer();
	setTimeout(() => {
		alert("game over !");
	}, 150);
};
</script>

<style>
@import "./assets/base.css";

#app {
	margin: 0 auto;
	padding: 2rem;
	text-align: center;
}

.mainContainer {
	width: fit-content;
	margin: auto;
	border: 5px rgb(230, 230, 230) outset;
	background-color: lightgray;
	padding: 15px;
}
.headContainer {
	border: 5px rgb(230, 230, 230) inset;
	padding: 15px;
	margin-bottom: 15px;
	display: flex;
	justify-content: space-between;
}
.headContainerItem {
	font-family: "Press Start 2P", cursive;
	height: 60px;
	background-color: black;
	color: red;
	font-size: 30px;
	display: flex;
	justify-content: center;
	align-items: center;
	padding: 5px;
}
.icon {
	transform: rotate(0.25turn);
	background-color: yellow;
	border-radius: 50%;
	color: black;
}
.icon:hover {
	cursor: pointer;
}
#gridContainer {
	border: 5px rgb(230, 230, 230) inset;
}
.row {
	display: flex;
}
</style>
