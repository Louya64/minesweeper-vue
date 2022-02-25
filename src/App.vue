<template>
	<h1>Démineur</h1>
	<div class="mainContainer">
		<div class="headContainer">
			<div class="headContainerItem nbMines">
				{{ nbMines < 100 ? "0" : "" }}{{ nbMines }}
			</div>
			<div class="headContainerItem">
				<p class="icon">{{ iconContent }}</p>
			</div>
			<div class="headContainerItem timer">
				{{ timer < 100 ? "0" : "" }}{{ timer < 10 ? "0" : "" }}{{ timer }}
			</div>
		</div>
		<div class="gridContainer">grille</div>
	</div>
</template>

<script setup lang="ts">
import { ref } from "vue";
const iconContent = ref(":(");
const nbMines = ref(10);
const timer = ref(0);
</script>

<style>
@import "./assets/base.css";

#app {
	margin: 0 auto;
	padding: 2rem;
	text-align: center;
}

.mainContainer {
	/* width: fit-content; */
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
}
.gridContainer {
	border: 5px rgb(230, 230, 230) inset;
}
</style>
