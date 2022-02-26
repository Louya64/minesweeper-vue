<template>
	<div class="cell">
		<div :class="valNumToColorClass">
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

// when props.neighbourEmpty changes => check if this cell is a neighbour and displays its content
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
				// to "tell" neighboors to check if they are empty too (props neighbourEmpty)
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
.classNonCree {
	transform: scale(1.5);
}
</style>
