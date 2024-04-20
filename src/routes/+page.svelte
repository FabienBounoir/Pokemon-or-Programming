<script>
	import { onMount } from 'svelte';
	import pokemon from '$lib/data/pokemon.json';
	import programming from '$lib/data/programming.json';

	/**
	 * @type {{name: string, url?: string, description?: string } | null}
	 */
	let data = null;

	let disabled = false;

	onMount(async () => {
		findTerm();
	});

	const findTerm = () => {
		const source = Math.random() > 0.5 ? pokemon : programming;
		let randomData = source[Math.floor(Math.random() * source.length)];

		if (data) {
			while (randomData.name.toLowerCase() == data.name.toLowerCase()) {
				console.log('randomData.name.toLowerCase()', randomData.name.toLowerCase());
				randomData = source[Math.floor(Math.random() * source.length)];
			}
		}

		data = randomData;
	};

	/**
	 *
	 * @param answer {string}
	 */
	const checkAnswer = (answer) => {
		disabled = true;
		if (
			(answer === 'Pokemon' && pokemon.includes(data)) ||
			(answer === 'Programming' && programming.includes(data))
		) {
			document.documentElement.style.setProperty('--background', '#CCFFCC');
		} else {
			document.documentElement.style.setProperty('--background', '#FFCCCC');
		}

		setTimeout(() => {
			findTerm();
			disabled = false;
		}, 1000);
	};

	/**
	 *
	 * @param type {string}
	 */
	const setPalette = (type) => {
		if (type === 'Pokemon') {
			document.documentElement.style.setProperty('--background', '#AEC6CF');
		} else if (type === 'Programming') {
			document.documentElement.style.setProperty('--background', '#B2EBCE');
		} else {
			document.documentElement.style.setProperty('--background', '#F5F5DC');
		}
	};

	const formatName = (name) => {
		return name
			.toLowerCase()
			.replace(/[^a-z0-9]/g, ' ')
			.split(' ')
			.map((word) => word.charAt(0).toUpperCase() + word.slice(1))
			.join(' ');
	};
</script>

<section>
	<h1>Pokemon or Programming</h1>

	{#if data}
		<div class="word-container">
			<h2>{formatName(data.name)}</h2>

			<div class="button-container">
				<button
					{disabled}
					id="pokemon"
					on:click={() => checkAnswer('Pokemon')}
					on:mouseover={() => setPalette('Pokemon')}
					on:mouseleave={() => setPalette('')}>Pokemon</button
				>
				<button
					{disabled}
					id="programming"
					on:click={() => checkAnswer('Programming')}
					on:mouseover={() => setPalette('Programming')}
					on:mouseleave={() => setPalette('')}>Programming</button
				>
			</div>
		</div>
	{/if}
</section>

<style lang="scss">
	section {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;

		h1 {
			margin-bottom: 20vh;
		}

		.word-container {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
		}

		button {
			color: white;
			border: none;
			padding: 10px 20px;
			margin: 10px;
			cursor: pointer;
			border-radius: 5px;

			&:disabled {
				cursor: not-allowed;
				opacity: 0.7;
			}
		}

		#pokemon {
			background-color: #4a707f;
			color: #ebf0f3;
		}

		#programming {
			background-color: #15865e;
			color: #d6f5e4;
		}
	}
</style>
