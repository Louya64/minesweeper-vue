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

const checkFlags = () => {
	//check is neighbours have flags
	// if nbflags === content => emit emptyZone
	if (content.value === 0) return;
	const grid = document.getElementById("gridContainer") as HTMLElement;
	let totalNeighbourFlags = 0;
	const cellsToCheck = [];

	// ex bdclick sur cell.pos[2,2] = la deuxième row 2ème col
	// cell -1,-1 = grid[0][0] => grid[pos[0]-2][pos[1]-2]
	if (grid.children[pos.value[0] - 2] !== undefined) {
		const cell1 = grid.children[pos.value[0] - 2]?.children[pos.value[1] - 2];
		// console.log("1", cell1);
		// console.log("drapeau", cell1.children[1].children[0]); // si drapeau, c'est pas undefined
		cellsToCheck.push(cell1);

		// cell -1,0 = grid[0][1] => grid[pos[0]-2][pos[1]-1]
		const cell2 = grid.children[pos.value[0] - 2].children[pos.value[1] - 1];
		// console.log("2", cell2);
		// console.log("pas drapeau", cell2.children[1].children[0]);
		cellsToCheck.push(cell2);

		// cell -1,+1 = grid[0][2] => grid[pos[0]-2][pos[1]]
		const cell3 = grid.children[pos.value[0] - 2].children[pos.value[1]];
		// console.log("3", cell3);
		cellsToCheck.push(cell3);
	}

	// cell 0,-1 = grid[1][0] => grid[pos[0]-1][pos[1]-2]
	const cell4 = grid.children[pos.value[0] - 1].children[pos.value[1] - 2];
	// console.log("4", cell4);
	cellsToCheck.push(cell4);

	// cell 0,+1 = grid[1][2] => grid[pos[0]-1][pos[1]]
	const cell5 = grid.children[pos.value[0] - 1].children[pos.value[1]];
	// console.log("5", cell5);
	cellsToCheck.push(cell5);

	if (grid.children[pos.value[0]] !== undefined) {
		// cell +1,-1 = grid[2][0] => grid[pos[0]][pos[1]-2]
		const cell6 = grid.children[pos.value[0]].children[pos.value[1] - 2];
		// console.log("6", cell6);
		cellsToCheck.push(cell6);

		// cell +1,0 = grid[2][1] => grid[pos[0]][pos[1]-1]
		const cell7 = grid.children[pos.value[0]].children[pos.value[1] - 1];
		// console.log("7", cell7);
		cellsToCheck.push(cell7);

		// cell +1,+1 = grid[2][2] => grid[pos[0]][pos[1]]
		const cell8 = grid.children[pos.value[0]].children[pos.value[1]];
		// console.log("8", cell8);
		cellsToCheck.push(cell8);
	}

	// cell already displayed = cell.children[1].classList.includes("displayNone")
	// cell with a flag = cell.children[1].children !== null ?
	cellsToCheck.map((cell) => {
		if (cell) {
			if (cell.children[1].children[0]) {
				totalNeighbourFlags++;
			}
		}
	});
	// console.log("totalNeighbourFlags", totalNeighbourFlags);
	// console.log("content", content.value);
	if (totalNeighbourFlags === content.value) {
		let loose = false;
		cellsToCheck.map((cell) => {
			if (cell) {
				if (cell.children[1].children[0] === undefined) {
					console.log(cell.children[0].classList);
					cell.children[1].classList.add("displayNone");
					if (cell.children[0].classList[0] === "bomb") {
						loose = true;
						console.log(cell);
						const img = document.createElement("img");
						img.src = "src/assets/croix.png";
						// img.classList.add("imgBombClicked"); // pk s'applique pas ???
						img.classList.add("imgToDelAtStart");
						img.style.position = "absolute";
						img.style.width = "40px";
						img.style.height = "40px";
						img.style.top = "-10px";
						img.style.left = "-8px";
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
watch(
	() => props.flags,
	(newVal) => {
		if (newVal === 10) {
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

<style scoped>
.cell {
	width: 50px;
	height: 50px;
	border: 2px solid rgb(143, 143, 143);
	display: flex;
	justify-content: center;
	align-items: center;
	font-size: 30px;
	font-family: "Press Start 2P", cursive;
	position: relative;
}
.hiding {
	position: absolute;
	width: 50px;
	height: 50px;
	background-color: blueviolet;
	border: 5px rgb(230, 230, 230) outset;
}
.displayNone {
	display: none;
}
.imgBombClicked {
	position: absolute;
	width: 40px;
	height: 40px;
	top: -10px;
	left: -8px;
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
	border: 3px solid black;
	color: transparent;
	width: 30px;
	height: 30px;
	margin: 10px;
}
</style>
