<script lang="ts">
	import { onMount } from 'svelte';

	let subtitlesTrack: HTMLTrackElement;

	let playerLeft: HTMLVideoElement;
	let playerRight: HTMLVideoElement;

	let leftDropzoneUploaded: boolean = false;
	let rightDropzoneUploaded: boolean = false;
	let subtitlesDropzoneUploaded: boolean = false;
	let videoPlayed: boolean = false;

	let dropzoneLeft: HTMLDivElement;
	let dropzoneRight: HTMLDivElement;
	let dropzoneSubtitles: HTMLDivElement;

	let inputLeft: HTMLInputElement;
	let inputRight: HTMLInputElement;
	let inputSubtitles: HTMLInputElement;

	let subtitleText: string = '';

	onMount(() => {
		// Dropzones
		function handleDrop(e: DragEvent, dropzone: HTMLDivElement, side: string) {
			e.preventDefault();
			dropzone.classList.remove('opacity-50');
			fileDrop(e, side);
		}

		function handleDragOver(e: DragEvent, dropzone: HTMLDivElement) {
			e.preventDefault();
			dropzone.classList.add('opacity-50');
		}

		function handleDragLeave(e: DragEvent, dropzone: HTMLDivElement) {
			e.preventDefault();
			dropzone.classList.remove('opacity-50');
		}

		// Left dropzone
		dropzoneLeft.addEventListener('drop', (e) => handleDrop(e, dropzoneLeft, 'left'));
		dropzoneLeft.addEventListener('dragover', (e) => handleDragOver(e, dropzoneLeft));
		dropzoneLeft.addEventListener('dragleave', (e) => handleDragLeave(e, dropzoneLeft));

		// Right dropzone
		dropzoneRight.addEventListener('drop', (e) => handleDrop(e, dropzoneRight, 'right'));
		dropzoneRight.addEventListener('dragover', (e) => handleDragOver(e, dropzoneRight));
		dropzoneRight.addEventListener('dragleave', (e) => handleDragLeave(e, dropzoneRight));

		// Subtitles dropzone
		dropzoneSubtitles.addEventListener('drop', (e) =>
			handleDrop(e, dropzoneSubtitles, 'subtitles')
		);
		dropzoneSubtitles.addEventListener('dragover', (e) => handleDragOver(e, dropzoneSubtitles));
		dropzoneSubtitles.addEventListener('dragleave', (e) => handleDragLeave(e, dropzoneSubtitles));

		// Inputs
		function handleInputChange(e: Event, side: string) {
			const input = e.target as HTMLInputElement;
			if (input.files) {
				const file = input.files[0];
				loadVideo(file, side);
			}
		}

		inputLeft.addEventListener('change', (e) => handleInputChange(e, 'left'));
		inputRight.addEventListener('change', (e) => handleInputChange(e, 'right'));
		inputSubtitles.addEventListener('change', (e) => handleInputChange(e, 'subtitles'));
	});

	function fileDrop(e: DragEvent, side: string) {
		if (e.dataTransfer?.items) {
			// If dropped items aren't files, reject them
			if (e.dataTransfer.items[0].kind === 'file') {
				let file = e.dataTransfer.items[0].getAsFile();
				if (!file) {
					throw new Error('No file found');
				}

				if (side === 'subtitles') {
					loadSubtitles(file);
				} else {
					loadVideo(file, side);
				}

				return;
			}
		}
	}

	function loadSubtitles(file: File) {
		const url = URL.createObjectURL(file);
		subtitlesTrack.src = url;
		subtitlesDropzoneUploaded = true;

		subtitlesTrack.track.oncuechange = (e) => {
			const cues = Array.from(subtitlesTrack.track.activeCues || []);
			//@ts-ignore
			const texts = cues ? [...cues].map((cue) => cue.text) : [];
			subtitleText = texts.join('\n');
		};
	}

	function loadVideo(file: File, side: string) {
		const url = URL.createObjectURL(file);
		let player: HTMLVideoElement | undefined;

		switch (side) {
			case 'left':
				player = playerLeft;
				leftDropzoneUploaded = true;

				playerLeft.addEventListener('play', () => {
					videoPlayed = true;
					if (playerRight) {
						playerRight.play();
						playerRight.currentTime = playerLeft.currentTime;
					}
				});

				playerLeft.addEventListener('pause', () => {
					if (playerRight) {
						playerRight.pause();
					}
				});

				playerLeft.addEventListener('seeked', () => {
					if (playerRight) {
						playerRight.currentTime = playerLeft.currentTime;
					}
				});

				break;
			case 'right':
				player = playerRight;
				rightDropzoneUploaded = true;
				break;
		}

		if (!player) {
			throw new Error('No player found');
		}

		player.src = url;
		player.load();
		player.classList.remove('hidden');
	}
</script>

<div class="grid grid-cols-2 items-center">
	<div
		class="width-full video-player flex items-center justify-center"
		class:border-r-2={!videoPlayed}
		bind:this={dropzoneLeft}
	>
		{#if leftDropzoneUploaded === false}
			<label
				class="mb-2 me-2 rounded-lg border border-gray-300 bg-white px-5 py-2.5 text-sm font-medium text-gray-900 hover:bg-gray-100 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:border-gray-600 dark:bg-gray-800 dark:text-white dark:hover:border-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-700"
			>
				Upload left video
				<input type="file" id="left-form" bind:this={inputLeft} class="hidden" />
			</label>
		{/if}
		<video controls bind:this={playerLeft} class="hidden">
			<track kind="metadata" default bind:this={subtitlesTrack} />
		</video>
	</div>

	<div class="width-full video-player flex items-center justify-center" bind:this={dropzoneRight}>
		{#if rightDropzoneUploaded === false}
			<label
				class="mb-2 me-2 rounded-lg border border-gray-300 bg-white px-5 py-2.5 text-sm font-medium text-gray-900 hover:bg-gray-100 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:border-gray-600 dark:bg-gray-800 dark:text-white dark:hover:border-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-700"
			>
				Upload right video
				<input type="file" id="right-form" bind:this={inputRight} class="hidden" />
			</label>
		{/if}
		<video bind:this={playerRight} muted class="hidden" />
	</div>
</div>

<div
	class="width-full subtitles flex items-center justify-center"
	class:border-t-2={!videoPlayed}
	bind:this={dropzoneSubtitles}
>
	{#if subtitlesDropzoneUploaded === false}
		<label
			class="mb-2 me-2 rounded-lg border border-gray-300 bg-white px-5 py-2.5 text-sm font-medium text-gray-900 hover:bg-gray-100 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:border-gray-600 dark:bg-gray-800 dark:text-white dark:hover:border-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-700"
		>
			Upload subtitles
			<input type="file" id="subtitles-form" bind:this={inputSubtitles} class="hidden" />
		</label>
	{/if}
	{#if subtitlesTrack?.track}
		<p class="bg-black/50 p-2 text-3xl">{subtitleText}</p>
	{/if}
</div>

<style>
	.video-player {
		height: 85vh;
	}
	.subtitles {
		height: 15vh;
	}
</style>
