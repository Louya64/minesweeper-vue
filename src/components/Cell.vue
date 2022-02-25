<template>
	<div class="cell">
		<div :class="valNumToColorClass">
			{{ content }}
		</div>
		<div
			@contextmenu="rightClick"
			:class="{ show: show }"
			@click="show = true"
			class="hidder"
		></div>
	</div>
</template>

<script setup lang="ts">
import { computed, watch, ref } from "vue";

interface Props {
	content: number;
}
const props = defineProps<Props>();
let content = ref(props.content);
const show = ref(false);

const rightClick = (e: Event) => {
	e.preventDefault();
	console.log("click droit");
};

watch(
	() => props.content,
	(newVal) => {
		console.log(newVal);
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
