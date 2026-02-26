<script>
	import { onDestroy, onMount } from 'svelte';
	import pokemon from '$lib/data/pokemon.json';
	import programming from '$lib/data/programming.json';
	import { scale } from 'svelte/transition';

	/**
	 * @typedef {'Pokemon' | 'Programming'} Category
	 */

	/**
	 * @typedef {{ name: string; url?: string; description?: string }} Entry
	 */

	/** @type {Entry | null} */
	let data = null;

	/** @type {Category | null} */
	let currentCategory = null;

	let disabled = false;
	let displayPrompt = true;
	let displayResponse = false;
	let gameOver = false;

	/** @type {number | null} */
	let feedbackTimeout = null;

	/** @type {number | null} */
	let nextRoundTimeout = null;

	/** @type {number | null} */
	let transitionTimeout = null;

	/** @type {(boolean | null)[]} */
	let results = [null, null, null, null, null];
	let currentRound = 0;
	const maxRounds = 5;

	/** @type {boolean | null} */
	let lastAnswerCorrect = null;

	onMount(() => {
		findTerm();
	});

	onDestroy(() => {
		clearTimers();
	});

	const clearTimers = () => {
		if (feedbackTimeout) {
			clearTimeout(feedbackTimeout);
			feedbackTimeout = null;
		}

		if (nextRoundTimeout) {
			clearTimeout(nextRoundTimeout);
			nextRoundTimeout = null;
		}

		if (transitionTimeout) {
			clearTimeout(transitionTimeout);
			transitionTimeout = null;
		}
	};

	/**
	 * @param {string} background
	 */
	const setBackground = (background) => {
		document.documentElement.style.setProperty('--background', background);
	};

	const findTerm = () => {
		currentCategory = Math.random() > 0.5 ? 'Pokemon' : 'Programming';
		const source = currentCategory === 'Pokemon' ? pokemon : programming;
		/** @type {Entry} */
		let randomData = source[Math.floor(Math.random() * source.length)];

		if (data) {
			while (randomData.name.toLowerCase() === data.name.toLowerCase()) {
				/** @type {Entry} */
				randomData = source[Math.floor(Math.random() * source.length)];
			}
		}

		data = randomData;
	};

	/**
	 * @param {Category} answer
	 */
	const checkAnswer = (answer) => {
		if (!data || !currentCategory || disabled || gameOver) return;

		clearTimers();
		disabled = true;
		displayPrompt = false;

		const isCorrect = answer === currentCategory;
		lastAnswerCorrect = isCorrect;
		results[currentRound] = isCorrect;

		transitionTimeout = setTimeout(() => {
			displayResponse = true;
			transitionTimeout = null;
		}, 220);

		feedbackTimeout = setTimeout(() => {
			setBackground(isCorrect ? '#59ff50' : '#ff9494');
		}, 300);
	};

	const nextRound = () => {
		if (!disabled) return;

		clearTimers();
		displayResponse = false;
		currentRound++;

		if (currentRound >= maxRounds) {
			gameOver = true;
			setPalette('');
			return;
		}

		transitionTimeout = setTimeout(() => {
			findTerm();
			displayPrompt = true;
			disabled = false;
			setPalette('');
			transitionTimeout = null;
		}, 220);
	};

	const restartGame = () => {
		clearTimers();
		results = [null, null, null, null, null];
		currentRound = 0;
		gameOver = false;
		displayResponse = false;
		displayPrompt = true;
		disabled = false;
		lastAnswerCorrect = null;
		findTerm();
		setPalette('');
	};

	/**
	 * @param {Category | ''} type
	 */
	const setPalette = (type) => {
		if (disabled) return;

		if (type === 'Pokemon') {
			setBackground('#fff886');
		} else if (type === 'Programming') {
			setBackground('#88dbff');
		} else {
			setBackground('#f6f6f6');
		}
	};

	/**
	 * @param {string} name
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
	<div class="title-wrap">
		<p class="eyebrow">Quick Interactive Quiz</p>
		<h1><span>Pokemon</span> or <span>Programming</span>?</h1>
		<p class="subtitle">Can you tell whether each term comes from Pokemon or from the coding world?</p>
	</div>

	<div class="score-container">
		{#each results as result, index}
			<div class="score-item" class:active={index === currentRound && !gameOver} class:correct={result === true} class:incorrect={result === false}>
				{#if result === true}
					<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M16.6667 5L7.5 14.1667L3.33334 10" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
				{:else if result === false}
					<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
				{:else}
					<span class="round-number">{index + 1}</span>
				{/if}
			</div>
		{/each}
	</div>

	<div class="card-stage" aria-live="polite">
		{#if data && displayPrompt}
			<div class="word-container" transition:scale={{ duration: 420 }}>
				<p class="card-label">Make your guess</p>
				<h2>{formatName(data.name)}</h2>

				<div class="button-container">
					<button
						{disabled}
						id="pokemon"
						on:click={() => checkAnswer('Pokemon')}
						on:mouseover={() => setPalette('Pokemon')}
						on:focus={() => setPalette('Pokemon')}
						on:mouseleave={() => setPalette('')}
						on:blur={() => setPalette('')}>Pokemon</button
					>
					<button
						{disabled}
						id="programming"
						on:click={() => checkAnswer('Programming')}
						on:mouseover={() => setPalette('Programming')}
						on:focus={() => setPalette('Programming')}
						on:mouseleave={() => setPalette('')}
						on:blur={() => setPalette('')}>Programming</button
					>
				</div>
			</div>
		{/if}

		{#if displayResponse}
			<div class="response" transition:scale={{ duration: 380 }}>
				{#if currentCategory === 'Pokemon'}
					<h2 class="response-title">Pokemon</h2>
					<img src={data?.url} alt={data?.name} loading="lazy" decoding="async" />
				{:else}
					<h2 class="response-title">Programming</h2>
					<p>{data?.description}</p>
				{/if}
				<button id="next" on:click={nextRound}>
					{currentRound < maxRounds - 1 ? 'Next Round' : 'See Results'}
				</button>
			</div>
		{/if}

		{#if gameOver}
			<div class="game-over" transition:scale={{ duration: 380 }}>
				<h2>Game Over!</h2>
				<p class="final-score">You scored <strong>{results.filter(r => r === true).length}</strong> out of <strong>{maxRounds}</strong></p>
				<button id="restart" on:click={restartGame}>Play Again</button>
			</div>
		{/if}
	</div>
</section>

<style lang="scss">
	section {
		position: relative;
		display: grid;
		grid-template-rows: auto 1fr;
		justify-items: center;
		align-items: start;
		min-height: 100dvh;
		padding: clamp(1.2rem, 2.4vw, 2rem) clamp(1rem, 2vw, 2rem) clamp(1.8rem, 3vw, 2.4rem);
		padding-top: clamp(3.2rem, 8vh, 6rem);
		gap: 1rem;
		overflow: hidden;

		&::before,
		&::after {
			content: '';
			position: absolute;
			width: clamp(220px, 38vw, 520px);
			height: clamp(220px, 38vw, 520px);
			border-radius: 50%;
			filter: blur(36px);
			z-index: 0;
			pointer-events: none;
		}

		&::before {
			background: radial-gradient(circle, rgba(255, 202, 40, 0.24) 0%, rgba(255, 202, 40, 0) 68%);
			top: -14%;
			left: -12%;
		}

		&::after {
			background: radial-gradient(circle, rgba(24, 148, 255, 0.24) 0%, rgba(24, 148, 255, 0) 68%);
			bottom: -16%;
			right: -10%;
		}

		> * {
			position: relative;
			z-index: 1;
		}

		.title-wrap {
			display: flex;
			flex-direction: column;
			gap: 0.5rem;
			align-items: center;
		}

		.eyebrow {
			margin: 0;
			padding: 0.35rem 0.8rem;
			border-radius: 999px;
			font-size: 0.78rem;
			font-weight: 800;
			text-transform: uppercase;
			letter-spacing: 0.06em;
			background: rgba(255, 255, 255, 0.72);
			border: 1px solid rgba(17, 24, 39, 0.1);
		}

		h1 {
			margin: 0;
			font-size: clamp(1.7rem, 3.2vw, 2.8rem);
			text-align: center;
			letter-spacing: 0.01em;
			line-height: 1.1;

			span:first-child {
				color: #b17c00;
			}

			span:last-child {
				color: #0b67b8;
			}
		}

		.subtitle {
			margin: 0;
			font-size: clamp(0.9rem, 1.5vw, 1.05rem);
			opacity: 0.86;
			text-align: center;
			text-wrap: balance;
			max-width: 52ch;
		}

		.card-stage {
			position: relative;
			width: min(92vw, 760px);
			min-height: clamp(260px, 38vh, 360px);
			align-self: center;
		}

		.word-container,
		.response {
			position: absolute;
			top: 0;
			left: 0;
			right: 0;
			width: 100%;
			min-height: 100%;
			padding: clamp(1.3rem, 2.9vw, 2.1rem);
			border-radius: 24px;
			background:
				linear-gradient(160deg, rgba(255, 255, 255, 0.9), rgba(247, 249, 255, 0.8));
			backdrop-filter: blur(10px);
			border: 1px solid rgba(255, 255, 255, 0.95);
			box-shadow:
				0 24px 56px rgba(17, 24, 39, 0.12),
				inset 0 1px 0 rgba(255, 255, 255, 0.92);
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			gap: 1rem;
            isolation: isolate;

			&::before {
				content: '';
				position: absolute;
				inset: 0;
				border-radius: inherit;
				background: linear-gradient(140deg, rgba(255, 255, 255, 0.22), rgba(255, 255, 255, 0));
				pointer-events: none;
				z-index: -1;
			}
        }

		.word-container {
			.card-label {
				margin: 0;
				font-size: 0.8rem;
				font-weight: 800;
				text-transform: uppercase;
				letter-spacing: 0.08em;
				opacity: 0.55;
			}

			h2 {
				margin: 0;
				text-align: center;
				font-size: clamp(1.9rem, 4.2vw, 3.2rem);
				line-height: 1.1;
				letter-spacing: 0.01em;
				text-wrap: balance;
			}
		}

		.button-container {
			display: flex;
			align-items: center;
			justify-content: center;
			flex-wrap: wrap;
			gap: 0.75rem;
		}

		.response {
			gap: 1rem;

			.response-title {
				margin: 0;
				font-size: clamp(1.15rem, 2.3vw, 1.55rem);
				text-transform: uppercase;
				letter-spacing: 0.09em;
				padding: 0.3rem 0.8rem;
				border-radius: 999px;
				background: rgba(17, 24, 39, 0.08);
			}

			img {
				image-rendering: pixelated;
				width: clamp(150px, 28vw, 250px);
				height: auto;
				filter: drop-shadow(0 10px 14px rgba(17, 24, 39, 0.18));
			}

			p {
				width: min(86vw, 540px);
				margin: 0;
				text-align: center;
				font-size: clamp(0.98rem, 1.5vw, 1.1rem);
				line-height: 1.6;
				opacity: 0.95;
				text-wrap: balance;
			}
		}

		button {
			min-width: clamp(160px, 24vw, 228px);
			font-size: clamp(1rem, 1.4vw, 1.1rem);
			font-weight: 800;
			letter-spacing: 0.02em;
			color: white;
			border: none;
			padding: 0.9rem 1.2rem;
			margin: 0;
			cursor: pointer;
			border-radius: 14px;
			transform: translateY(0);
			transition:
				transform 0.18s ease,
				box-shadow 0.2s ease,
				background-color 0.25s ease,
				color 0.25s ease,
				filter 0.2s ease;
			box-shadow: 0 10px 20px rgba(17, 24, 39, 0.14);

			&:hover,
			&:focus-visible {
				transform: translateY(-3px) scale(1.01);
				box-shadow: 0 14px 30px rgba(17, 24, 39, 0.2);
				filter: saturate(1.1);
			}

			&:focus-visible {
				outline: 3px solid rgba(17, 24, 39, 0.25);
				outline-offset: 2px;
			}

			&:disabled {
				cursor: not-allowed;
				opacity: 0.65;
				transform: none;
				filter: grayscale(0.08);
				box-shadow: 0 6px 14px rgba(17, 24, 39, 0.1);
			}
		}

		#pokemon {
			background: linear-gradient(145deg, #ffd84f, #ffbc00);
			color: #442304;

			&:hover {
				color: white;
				background: linear-gradient(145deg, #f4b800, #cb8f00);
			}
		}

		#programming {
			background: linear-gradient(145deg, #4fb4ff, #1894ff);
			color: #11305a;

			&:hover {
				color: white;
				background: linear-gradient(145deg, #2199ff, #0c77d3);
			}
		}

		#next {
			margin-top: 0.5rem;
			background: linear-gradient(145deg, #4ade80, #22c55e);
			color: #052e16;

			&:hover {
				color: white;
				background: linear-gradient(145deg, #22c55e, #16a34a);
			}
		}

		#restart {
			margin-top: 0.5rem;
			background: linear-gradient(145deg, #60a5fa, #3b82f6);
			color: #1e3a8a;

			&:hover {
				color: white;
				background: linear-gradient(145deg, #3b82f6, #2563eb);
			}
		}

		.score-container {
			position: fixed;
			right: clamp(1rem, 3vw, 2.5rem);
			top: 50%;
			transform: translateY(-50%);
			display: flex;
			flex-direction: column;
			gap: 0.65rem;
			z-index: 10;
		}

		.score-item {
			width: 48px;
			height: 48px;
			border-radius: 12px;
			display: flex;
			align-items: center;
			justify-content: center;
			background: rgba(255, 255, 255, 0.85);
			border: 2px solid rgba(17, 24, 39, 0.1);
			box-shadow: 0 4px 12px rgba(17, 24, 39, 0.08);
			transition: all 0.3s ease;
			font-weight: 700;
			font-size: 0.95rem;
			color: rgba(17, 24, 39, 0.4);

			&.active {
				border-color: rgba(59, 130, 246, 0.5);
				box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15), 0 4px 12px rgba(17, 24, 39, 0.12);
				transform: scale(1.08);
			}

			&.correct {
				background: linear-gradient(145deg, #4ade80, #22c55e);
				border-color: #16a34a;
				color: white;

				svg {
					stroke-width: 3;
				}
			}

			&.incorrect {
				background: linear-gradient(145deg, #f87171, #ef4444);
				border-color: #dc2626;
				color: white;

				svg {
					stroke-width: 3;
				}
			}

			.round-number {
				opacity: 0.6;
			}
		}

		.game-over {
			position: absolute;
			top: 0;
			left: 0;
			right: 0;
			width: 100%;
			min-height: 100%;
			padding: clamp(1.3rem, 2.9vw, 2.1rem);
			border-radius: 24px;
			background:
				linear-gradient(160deg, rgba(255, 255, 255, 0.9), rgba(247, 249, 255, 0.8));
			backdrop-filter: blur(10px);
			border: 1px solid rgba(255, 255, 255, 0.95);
			box-shadow:
				0 24px 56px rgba(17, 24, 39, 0.12),
				inset 0 1px 0 rgba(255, 255, 255, 0.92);
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			gap: 1.5rem;
			isolation: isolate;

			&::before {
				content: '';
				position: absolute;
				inset: 0;
				border-radius: inherit;
				background: linear-gradient(140deg, rgba(255, 255, 255, 0.22), rgba(255, 255, 255, 0));
				pointer-events: none;
				z-index: -1;
			}

			h2 {
				margin: 0;
				font-size: clamp(2rem, 4vw, 2.8rem);
				text-align: center;
				letter-spacing: 0.02em;
			}

			.final-score {
				margin: 0;
				font-size: clamp(1.1rem, 2vw, 1.3rem);
				text-align: center;

				strong {
					color: #3b82f6;
					font-size: 1.4em;
				}
			}
		}

		@media (max-width: 768px) {
			.score-container {
				position: fixed;
				bottom: clamp(1rem, 3vh, 2rem);
				top: auto;
				left: 50%;
				right: auto;
				transform: translateX(-50%);
				flex-direction: row;
				gap: 0.6rem;
			}

			.score-item {
				width: 44px;
				height: 44px;
				font-size: 0.9rem;
			}
		}

		@media (max-width: 560px) {
			.title-wrap {
				gap: 0.35rem;
			}

			.card-stage {
				min-height: clamp(280px, 44vh, 380px);
			}

			.button-container {
				flex-direction: column;
				width: 100%;
			}

			button {
				width: 100%;
			}

			.score-container {
				bottom: 0.75rem;
				gap: 0.5rem;
			}

			.score-item {
				width: 40px;
				height: 40px;
				font-size: 0.85rem;
			}
		}

		@media (prefers-reduced-motion: reduce) {
			* {
				animation: none !important;
				transition-duration: 0s !important;
			}
		}
	}
</style>
