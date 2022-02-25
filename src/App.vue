<template>
	<h1>Démineur</h1>
	<button @click="start">Start</button>
	<div class="mainContainer">
		<div class="headContainer">
			<div class="headContainerItem nbMines">
				{{ nbMines < 100 ? "0" : "" }}{{ nbMines }}
			</div>
			<div @click="start" class="headContainerItem">
				<p class="icon">{{ iconContent }}</p>
			</div>
			<div class="headContainerItem timer">
				{{ timer < 100 ? "0" : "" }}{{ timer < 10 ? "0" : "" }}{{ timer }}
			</div>
		</div>
		<div class="gridContainer">
			<div class="row" v-for="(row, indexRow) in rows">
				<Cell
					:content="gridContent[indexRow][indexCol]"
					v-for="(col, indexCol) in cols"
				/>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import Cell from "./components/Cell.vue";
import { ref, reactive } from "vue";
const iconContent = ref(":)"); // début :) / perdu :( / gagné :D
const nbMines = ref(10);
const timer = ref(0);
const rows = ref(9);
const cols = ref(9);
// const content = ref(10);
let gridContent = reactive([] as number[][]);

for (let i = 0; i < rows.value; i++) {
	gridContent.push([]);
	for (let j = 0; j < cols.value; j++) {
		gridContent[i].push(0);
	}
}

const start = () => {
	gridContent.map(
		(row, rowIndex) => (gridContent[rowIndex] = row.map(() => 0))
	);
	randomFillBombs();
};

const calculate = () => {
	gridContent.map((row, indexRow) =>
		row.map((col, indexCol) => {
			if (gridContent[indexRow][indexCol] === 10) {
				// ligne du dessus de gauche à droite
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

				//m^meme ligne de gauche à droite
				if (indexCol - 1 !== -1 && gridContent[indexRow][indexCol - 1] !== 10) {
					gridContent[indexRow][indexCol - 1] += 1;
				}
				if (
					indexCol + 1 !== cols.value &&
					gridContent[indexRow][indexCol + 1] !== 10
				) {
					gridContent[indexRow][indexCol + 1] += 1;
				}

				// ligne de dessous de gauche à droite
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
	console.log(gridContent);
};

const randomFillBombs = () => {
	for (let i = 0; i < nbMines.value; i++) {
		const randomRow = Math.floor(Math.random() * rows.value);
		const randomCol = Math.floor(Math.random() * cols.value);
		if (gridContent[randomRow][randomCol] === 10) {
			i--;
		}
		gridContent[randomRow][randomCol] = 10;
	}
	calculate();
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
.gridContainer {
	border: 5px rgb(230, 230, 230) inset;
}
.row {
	display: flex;
}
</style>
