<template>
	<div class="cell">
		<div @dblclick="checkFlags" :class="valNumToColorClass">
			<img
				class="imgBombClicked"
				v-if="bombClicked"
				src="../assets/croix.png"
				alt=""
			/>
			{{ content }}
		</div>
		<div
			@contextmenu="rightClick"
			:class="{ displayNone: showCell }"
			@click="leftClick"
			class="hiding"
		>
			<p v-if="flag">!</p>
		</div>
	</div>
</template>

<script setup lang="ts">
import { computed, watch, ref } from "vue";

interface Props {
	isStarted: boolean;
	flags: number;
	nbMines: number;
	rows: number;
	cols: number;
	pos: number[];
	content: number;
	show: boolean;
	neighbourEmpty: number[];
	gridContent: number[][];
}
const emit = defineEmits<{
	(e: "game-over"): void;
	(e: "emptyZone", pos: number[]): void;
	(e: "resetEmptyZone"): void;
	(e: "flagUsed", nb: number): void;
	(e: "checkWin"): void;
}>();

const props = defineProps<Props>();
const pos = ref(props.pos);
let content = ref(props.content);
let showCell = ref(props.show);
let totalMines = ref(props.nbMines);

let bombClicked = ref(false); // displays a cross if content = bomb and cell is clicked (to know where you have 'failed' when all is displayed)
const flag = ref(false); // displays a flag when true

// emptyZone has been emitted => neighbourEmpty changes => check if this cell is a neighbour and displays its content
watch(
	() => props.neighbourEmpty,
	(newVal) => {
		// return if newVal is itself
		if (newVal === [props.rows + 10, props.cols + 10]) return;
		// return if content is already displayed
		if (showCell.value) return;

		//utile?
		if (
			newVal[0] - 1 < 0 ||
			newVal[0] + 1 > props.rows + 1 ||
			newVal[1] - 1 < 0 ||
			newVal[1] + 1 > props.cols + 1
		)
			return;

		//si cette cell est voisine d'une qui a émit emptyZone
		if (
			(pos.value[0] === newVal[0] - 1 && pos.value[1] === newVal[1] - 1) ||
			(pos.value[0] === newVal[0] - 1 && pos.value[1] === newVal[1]) ||
			(pos.value[0] === newVal[0] - 1 && pos.value[1] === newVal[1] + 1) ||
			(pos.value[0] === newVal[0] && pos.value[1] === newVal[1] - 1) ||
			(pos.value[0] === newVal[0] && pos.value[1] === newVal[1] + 1) ||
			(pos.value[0] === newVal[0] + 1 && pos.value[1] === newVal[1] - 1) ||
			(pos.value[0] === newVal[0] + 1 && pos.value[1] === newVal[1]) ||
			(pos.value[0] === newVal[0] + 1 && pos.value[1] === newVal[1] + 1)
		) {
			// et si c'est pas une bomb
			if (content.value !== 10) {
				leftClick();
			}
		}
	}
);

// on dbClick on a numberer visible cell
const checkFlags = () => {
	//check is neighbours have flags
	// if nbflags === this cell content => display neighbours content
	if (content.value === 0) return;
	const grid = document.getElementById("gameGrid") as HTMLElement;
	let totalNeighbourFlags = 0;
	const cellsToCheck = [];

	// row above
	if (grid.children[pos.value[0] - 2] !== undefined) {
		const cell1 = grid.children[pos.value[0] - 2]?.children[pos.value[1] - 2];
		cellsToCheck.push(cell1);

		const cell2 = grid.children[pos.value[0] - 2].children[pos.value[1] - 1];
		cellsToCheck.push(cell2);

		const cell3 = grid.children[pos.value[0] - 2].children[pos.value[1]];
		cellsToCheck.push(cell3);
	}

	// same row
	const cell4 = grid.children[pos.value[0] - 1].children[pos.value[1] - 2];
	cellsToCheck.push(cell4);

	const cell5 = grid.children[pos.value[0] - 1].children[pos.value[1]];
	cellsToCheck.push(cell5);

	// row below
	if (grid.children[pos.value[0]] !== undefined) {
		const cell6 = grid.children[pos.value[0]].children[pos.value[1] - 2];
		cellsToCheck.push(cell6);

		const cell7 = grid.children[pos.value[0]].children[pos.value[1] - 1];
		cellsToCheck.push(cell7);

		const cell8 = grid.children[pos.value[0]].children[pos.value[1]];
		cellsToCheck.push(cell8);
	}

	// cell.children[1].children[0] = there is a flag (else, cell.children[1].children[0] = undefined)
	cellsToCheck.map((cell) => {
		if (cell) {
			if (cell.children[1].children[0]) {
				totalNeighbourFlags++;
			}
		}
	});

	if (totalNeighbourFlags === content.value) {
		let loose = false;
		cellsToCheck.map((cell) => {
			if (cell) {
				// if there is no flag
				if (cell.children[1].children[0] === undefined) {
					cell.children[1].classList.add("displayNone");

					if (cell.children[0].classList[0] === "bomb") {
						loose = true;
						console.log(cell);
						const img = document.createElement("img");
						img.src = "src/assets/croix.png";
						img.classList.add("imgBombClicked");
						img.classList.add("imgToDelAtStart");
						cell.children[0].append(img);
						setTimeout(() => {
							emit("game-over");
						}, 250);
					}
				}
			}
		});
		if (!loose) {
			emit("checkWin");
		}
	}
};

// display the content of the cell
const leftClick = () => {
	// disable when there is a flag or game is not started
	if (flag.value || !props.isStarted) return;
	emit("resetEmptyZone");
	showCell.value = true;
	setTimeout(() => {
		// content = 10 = bomb
		if (content.value === 10) {
			bombClicked.value = true;
			setTimeout(() => {
				emit("game-over");
			}, 250);
		} else {
			emit("checkWin");
			// content = 0 = empty
			if (content.value === 0) {
				// to "tell" neighboors to check if they are empty too (watch props neighbourEmpty)
				emit("emptyZone", pos.value);
			}
		}
	}, 1);
};

// add a flag ... or not
const rightClick = (e: Event) => {
	e.preventDefault();
	// prevent to add more flags than total bombs
	if (props.flags === 0 && !flag.value) return;

	// toggle flag
	flag.value = !flag.value;

	// send -1 if flag was added or 1 if removed to calculate flags remaining
	let nb: number;
	flag.value ? (nb = -1) : (nb = 1);
	emit("flagUsed", nb);
};

// reset "show" when game starts(props.show passed to false) (remove cross img, put back hiding div)
watch(
	() => props.show,
	(newVal, oldVal) => {
		if (!newVal) {
			bombClicked.value = false; // img
			showCell.value = newVal;
		}
		return (showCell.value = newVal);
	}
);

//
watch(
	() => props.content,
	(newVal) => {
		return (content.value = newVal);
	}
);

// reset flags displayed when game starts
// must first listen to nbMines props changing
watch(
	() => props.nbMines,
	(newVal) => {
		return (totalMines.value = newVal);
	}
);
watch(
	() => props.flags,
	(newVal) => {
		if (newVal === totalMines.value) {
			return (flag.value = false);
		}
	}
);

// change color (css) depending on content
const valNumToColorClass = computed(() => {
	switch (props.content) {
		case 0:
			return "invisible";
		case 1:
			return "one";
		case 2:
			return "two";
		case 3:
			return "three";
		case 4:
			return "four";
		case 5:
			return "five";
		case 6:
			return "six";
		case 7:
			return "seven";
		case 8:
			return "eight";
		case 10:
			return "bomb";
		default:
			return "unknown";
	}
});
</script>

<style>
.imgBombClicked {
	position: absolute;
	width: 35px;
	height: 35px;
	top: -0.5vw;
	left: -0.5vw;
}
@media all and (max-width: 1224px) {
	.imgBombClicked {
		width: 30px;
		height: 30px;
	}
}
</style>

<style scoped>
.cell,
.hiding {
	width: 45px;
	height: 45px;
}
.cell {
	border: 1px solid rgb(143, 143, 143);
	display: flex;
	justify-content: center;
	align-items: center;
	font-size: 25px;
	font-family: "Press Start 2P", cursive;
	position: relative;
}
.hiding {
	position: absolute;
	background-color: blueviolet;
	border: 4px rgb(230, 230, 230) outset;
}
@media all and (max-width: 1224px) {
	.cell,
	.hiding {
		width: 35px;
		height: 35px;
		font-size: 22px;
	}
}
.displayNone {
	display: none;
}
.invisible {
	color: transparent;
}
.one {
	color: rgb(57, 57, 250);
}
.two {
	color: green;
}
.three {
	color: rgba(255, 0, 0, 0.746);
}
.four {
	color: rgb(0, 0, 144);
}
.five {
	color: rgb(138, 38, 38);
}
.six {
	color: rgb(54, 165, 154);
}
.seven {
	color: rgb(109, 246, 109);
}
.eight {
	color: rgb(236, 240, 36);
}
.bomb {
	border-radius: 50%;
	background-color: red;
	border: 0.2vw solid black;
	color: transparent;
	width: 27px;
	height: 27px;
	margin: 10px;
}
@media all and (max-width: 1224px) {
	.bomb {
		width: 22px;
		height: 22px;
	}
}
</style>
