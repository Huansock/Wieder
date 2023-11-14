<script>
	import { onMount, tick } from 'svelte';
	let recognition;
	let transcriptSource = 'Source for Transcripts';
	$: transcripts = [];
	let interim = 'interim';
	let targetLang;
	let errorMessage;

	onMount(async () => {
		let SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
		recognition = new SpeechRecognition();
		recognition.onstart = (event) => {
			console.log('started');
			console.log('hi');
		};
		recognition.onend = (event) => {
			recognition.start();
		};
		// recognition onresult setting
		recognition.onresult = async (event) => {
			let isfinal = false;
			for (let i = 0; i < event.results.length; i++) {
				let res = event.results[i];
				let trans = res[0].transcript;
				if (!res.isFinal) {
					transcriptSource = trans;
				} else {
					console.log('isfinal');
					transcriptSource = trans;
					isfinal = true;
					//currentTime setting
					const currentTime = new Date().toLocaleTimeString();
					transcripts = [...transcripts, `[${currentTime}]   ${trans}`];
					console.log(transcripts);
				}
			}
		};
		recognition.onerror = function (event) {
			errorMessage = 'Error occurred in recognition: ' + event.error;
			console.log(event);
		};
	});

	function setAttribute() {
		recognition.lang = targetLang;
		//recognition attributes
		recognition.continuous = false;
		recognition.interimResults = true;
		recognition.maxAlternatives = 1;
		console.log('attribute set');
	}

	function startButton() {
		recognition.start();
	}
</script>

<h1>Live transcript webspeech api</h1>
<p>
	<input
		type="text"
		bind:value={targetLang}
		placeholder="which language do you want?"
		style="min-width: 20rem; min-height:5rem;"
	/>
	<button on:click={setAttribute}>set attribute</button>
</p>
<button on:click={startButton}>start button</button>
<p>{transcriptSource}</p>

<div class="previous-scripts">
	{#each transcripts.reverse() as transcript}
		<p>{transcript}</p>
	{/each}
</div>

<p>{errorMessage}</p>

<style>
	.previous-scripts {
		color: dimgrey;
	}
</style>
