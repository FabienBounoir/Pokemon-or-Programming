<script>
	import { onMount } from 'svelte';
	import pokemon from '$lib/data/pokemon.json';
	import programming from '$lib/data/programming.json';
	import { scale } from 'svelte/transition';

	/**
	 * @type {{name: string, url?: string, description?: string } | null}
	 */
	let data = null;

	let disabled = false;

	let displayResponse = false;

	onMount(async () => {
		findTerm();
	});

	const findTerm = () => {
		const source = Math.random() > 0.5 ? pokemon : programming;
		let randomData = source[Math.floor(Math.random() * source.length)];

		if (data) {
			while (randomData.name.toLowerCase() == data.name.toLowerCase()) {
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
		displayResponse = true;
		setTimeout(() => {
			if (
				(answer === 'Pokemon' && pokemon.includes(data)) ||
				(answer === 'Programming' && programming.includes(data))
			) {
				document.documentElement.style.setProperty('--background', '#59ff50');
			} else {
				document.documentElement.style.setProperty('--background', '#ff9494');
			}
		}, 400);

		setTimeout(() => {
			disabled = false;
			setPalette('');
			findTerm();
			displayResponse = false;
		}, 6000);
	};

	/**
	 *
	 * @param type {string}
	 */
	const setPalette = (type) => {
		if (disabled) return;

		if (type === 'Pokemon') {
			document.documentElement.style.setProperty('--background', '#fff886');
		} else if (type === 'Programming') {
			document.documentElement.style.setProperty('--background', '#88dbff');
		} else {
			document.documentElement.style.setProperty('--background', '#f6f6f6');
		}
	};

	/**
	 *
	 * @param name {string}
	 */
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
	<h1>Pokemon or Programming ?</h1>

	{#if data && !displayResponse}
		<div class="word-container" transition:scale={{ duration: 599 }}>
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

	{#if displayResponse}
		<div class="response" in:scale={{ delay: 600 }}>
			{#if data?.url}
				<h2>Pokemon</h2>
				<img src={data?.url} alt={data?.name} />
			{:else}
				<h2>Programming</h2>
				<p>{data?.description}</p>
			{/if}
		</div>
	{/if}
</section>

<style lang="scss">
	section {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 98dvh;

		h1 {
			position: absolute;
			top: 0;
		}

		.word-container {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
		}

		.response {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;

			h2 {
				padding: 0;
				margin: 0;
			}

			img {
				image-rendering: pixelated;
				width: 200px;
			}

			p {
				width: min(98vw, 500px);
				text-align: center;
				text-wrap: balance;
			}
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
			background-color: #ffcc00;
			transition:
				background-color 0.3s,
				color 0.3s;
			color: #442304;

			&:hover {
				color: white;
				background-color: #d19500;
			}
		}

		#programming {
			background-color: #28a7ff;
			transition:
				background-color 0.3s,
				color 0.3s;
			color: #11305a;

			&:hover {
				color: white;
				background-color: #1e90ff;
			}
		}
	}
</style>
