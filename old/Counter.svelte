<script lang="ts">
	import { getCookie } from './Cookie.svelte';
	import { setCookie } from './Cookie.svelte';

	let count = 1;

	function loadValFromCookie() {
		count = Number(getCookie("count"));
		if (count == "") count = 0;
	}
	function handleClick() {
		count += 1;
		setCookie("count", count, 1);
	}
	function factorial(n) {
		if (n <= 1)
			return 1;
		else return n * factorial(n - 1);
	}
	loadValFromCookie();
</script>

<button class="counter" on:click={handleClick}>
	clicks: {count + Math.round(Math.random() * count / 4 - count / 8)}
</button>

<style>
	.counter {
		border:  2px solid black;
		background: white;
		color: black;
	}
</style>