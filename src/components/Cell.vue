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
			:class="{ show: show }"
			@click="leftClick"
			class="hidder"
		></div>
	</div>
</template>

<script setup lang="ts">
import { computed, watch, ref } from "vue";

interface Props {
	content: number;
	show: boolean;
	pos: number[];
}
const emit = defineEmits<{
	(e: "game-over"): void;
	(e: "emptyZone", pos: number[]): void;
}>();

const props = defineProps<Props>();
let content = ref(props.content);
let show = ref(props.show);
let bombClicked = ref(false);
const pos = ref(props.pos);

const leftClick = () => {
	show.value = true;
	if (content.value === 0) {
		// découvrir la zone vide
		emit("emptyZone", pos.value);
	}
	if (content.value === 10) {
		bombClicked.value = true;
		emit("game-over");
	}
};

const rightClick = (e: Event) => {
	e.preventDefault();
	console.log("click droit");
};

watch(
	() => props.show,
	(newVal) => {
		if (!newVal) {
			bombClicked.value = false;
		}

		return (show.value = newVal);
	}
);

watch(
	() => props.content,
	(newVal) => {
		return (content.value = newVal);
	}
);

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
		case 10:
			return "bomb";
		default:
			return "classNonCree";
	}
});
</script>

<style>
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
.hidder {
	position: absolute;
	width: 50px;
	height: 50px;
	background-color: blueviolet;
	border: 5px rgb(230, 230, 230) outset;
}
.show {
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
