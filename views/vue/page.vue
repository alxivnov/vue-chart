<template>
	<div>
		<h1>{{ title }}</h1>
		<btn @click="increment">++</btn>
	</div>
</template>

<script>
export default {
	props: {
		startIndex: Number
	},
	data() {
		return {
			temp: 'Page ',
			numClicks: 0
		}
	},
	computed: {
		title() {
			return this.temp + (this.startIndex + this.numClicks);
		}
	},
	methods: {
		increment() {
			this.numClicks++;
		}
	}
}
</script>