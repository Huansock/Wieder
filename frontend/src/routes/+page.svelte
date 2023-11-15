<script>
	import { onMount, tick } from 'svelte';
	import { saveAs } from 'file-saver';
	let recognition;
	let transcriptSource = 'Source for Transcripts';
	$: transcripts = [];
	$: reversedTranscripts = [];
	let targetLang;
	let errorMessage;
	let Time4FileName = new Date().toLocaleTimeString();

	onMount(async () => {
		let SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
		recognition = new SpeechRecognition();
		recognition.onstart = (event) => {
			console.log('started');
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
					transcripts.push(`[${currentTime}]   ${trans} \n`);
					transcripts = transcripts;
					reversedTranscripts = transcripts.toReversed();
					console.log(transcripts);
					console.log(reversedTranscripts);
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
		recognition.maxAlternatives = 100;
		console.log('attribute set');
	}

	function startButton() {
		recognition.start();
	}
	function CreateTextFile() {
		let blob = new Blob(transcripts, {
			type: 'text/plain;charset=utf-8',
			endings: 'native'
		});
		saveAs(blob, `${Time4FileName}.txt`);
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
<button on:click={CreateTextFile}>create Txt file</button>
<p>{transcriptSource}</p>

<div class="previous-scripts">
	{#each reversedTranscripts as transcript}
		<p>{transcript}</p>
	{/each}
</div>

<p>{errorMessage}</p>

<style>
	.previous-scripts {
		color: dimgrey;
	}
</style>
