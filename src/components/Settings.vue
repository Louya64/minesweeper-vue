<template>
	<div class="setting">
		<div class="settingsItem">
			<label for="width">Changer la hauteur</label>
			<input type="number" id="width" v-model="rows" />
		</div>
		<div class="settingsItem">
			<label for="height">Changer la largeur</label>
			<input type="number" id="height" v-model="cols" />
		</div>
		<div class="settingsItem">
			<label for="bombs">Changer les bombes</label>
			<input type="number" id="bombs" v-model="nbMines" />
		</div>
	</div>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";

const emit = defineEmits<{
	(e: "newRows", nb: number): void;
	(e: "newCols", nb: number): void;
	(e: "newNbMines", nb: number): void;
}>();

const rows = ref(9);
const cols = ref(9);
const nbMines = ref(10);

watch(
	() => rows.value,
	(newVal) => {
		emit("newRows", newVal);
	}
);
watch(
	() => cols.value,
	(newVal) => {
		emit("newCols", newVal);
	}
);
watch(
	() => nbMines.value,
	(newVal) => {
		emit("newNbMines", newVal);
	}
);
</script>

<style scoped>
.setting {
	z-index: 1;
	position: absolute;
	top: 10vw;
	left: 0;
	padding: 2vw;
	text-align: left;
	background-color: rgb(242, 237, 246);
}

.settingsItem {
	padding: 0.5vw;
	width: 20vw;
	border-bottom: 1px solid rgba(0, 0, 0, 0.185);
	display: flex;
	justify-content: space-between;
}

.settingsItem input {
	width: 3vw;
	border: none;
	text-align: center;
	background-color: transparent;
}
</style>
