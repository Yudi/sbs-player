<script lang="ts">
	import { onMount } from 'svelte';

	onMount(() => {
		// @ts-ignore
		window.SUPPORTS_SKIP_SILENCE_SHADOW_PLAYER = true;

		const playSelectedFile = (file: File, subtitle: File) => {
			const videoNode = document.getElementById('player') as HTMLVideoElement;
			const subtitleTrack = document.getElementById('subtrack') as HTMLTrackElement;
			const fileURL = window.URL.createObjectURL(file);
			const subtitleURL = window.URL.createObjectURL(subtitle);

			if (!videoNode) {
				throw new Error('No video node found');
			}

			if (!subtitleTrack) {
				throw new Error('No subtitle track found');
			}

			videoNode.src = fileURL;
			subtitleTrack.src = subtitleURL;

			document.querySelector('.selection')?.classList.add('hidden');
			document.querySelector('#player')?.classList.remove('hidden');
			document.querySelector('.activate-info')?.classList.remove('hidden');

			const dragInfo = document.querySelector('.drag-info') as HTMLElement;
			dragInfo.style.display = 'none';
		};

		// Handle selecting a new file
		const inputNode = document.getElementById('input') as HTMLInputElement;
		if (!inputNode) {
			throw new Error('No input node found');
		}

		inputNode.addEventListener(
			'change',
			(evt) => {
				const input = evt.target as HTMLInputElement;
				if (input.files) {
					const files = Array.from(input.files);
					const videoFile = files.find((file) => file.type.startsWith('video/'));
					const subtitleFile = files.find((file) => file.type.startsWith('text/'));
					if (!videoFile) {
						throw new Error('No video file found');
					}
					if (!subtitleFile) {
						throw new Error('No subtitle file found');
					}

					playSelectedFile(videoFile, subtitleFile);
				}
			},
			false
		);

		// // Handle Drag and Drop of files over the page
		// document.body.addEventListener('dragover', (ev) => {
		// 	ev.preventDefault();
		// 	(document.querySelector('.drag-info') as HTMLElement).style.display = 'flex';
		// });
		// document.body.addEventListener('dragexit', () => {
		// 	(document.querySelector('.drag-info') as HTMLElement).style.display = 'none';
		// });
		// document.body.addEventListener('drop', (ev) => {
		// 	console.log('File(s) dropped');

		// 	// Prevent default behavior (Prevent file from being opened)
		// 	ev.preventDefault();
		// 	if (!ev.dataTransfer) {
		// 		throw new Error('No data transfer found');
		// 	}

		// 	if (ev.dataTransfer.items) {
		// 		// Use DataTransferItemList interface to access the file(s)
		// 		for (var i = 0; i < ev.dataTransfer.items.length; i++) {
		// 			// If dropped items aren't files, reject them
		// 			if (ev.dataTransfer.items[i].kind === 'file') {
		// 				var file = ev.dataTransfer.items[i].getAsFile();

		// 				playSelectedFile(file);
		// 				return;
		// 			}
		// 		}
		// 	} else {
		// 		// Use DataTransfer interface to access the file(s)

		// 		for (var i = 0; i < ev.dataTransfer.files.length; i++) {
		// 			playSelectedFile(ev.dataTransfer.files[i]);
		// 			return;
		// 		}
		// 	}
		// });
	});
</script>

<div class="activate-info hidden">
	<!-- <img src="arrow.svg" alt="Arrow" /> -->
	<p>You can now activate "Skip Silence"</p>
</div>

<div class="drag-info hidden">
	<p>Drop your video to play it</p>
</div>

<div class="selection">
	<h1>Skip Silence Video Player</h1>
	<p>
		This player allows you to use the <a
			href="https://chrome.google.com/webstore/detail/skip-silence/fhdmkhbefcbhakffdihhceaklaigdllh"
			>"Skip Silence" extension</a
		> in combination with local files.
	</p>
	<p>Simply choose the video you want to watch, then activate "Skip Silence" as normal.</p>

	<div class="file">
		<label class="fileContainer">
			Choose a video
			<input type="file" accept="video/*, .vtt" id="input" multiple />
		</label>
	</div>
</div>
<video controls autoplay id="player" class="hidden">
	<track label="Portuguese" kind="subtitles" srclang="pt" id="subtrack" default />
</video>

<style>
	a {
		color: #d1d1d1;
	}

	.hidden {
		display: none;
	}

	.activate-info {
		position: absolute;
		top: 0;
		right: 0;

		animation-name: fadeInOut;
		animation-duration: 4s;
		animation-fill-mode: forwards;

		background-color: rgba(0, 0, 0, 0.8);
		padding: 1em 6em;

		opacity: 0;
	}

	.drag-info {
		position: absolute;
		top: 0;
		left: 0;

		background-color: rgba(0, 0, 0, 0.8);
		width: 100%;
		height: 100%;

		justify-content: center;
		align-items: center;
	}

	@keyframes fadeInOut {
		0% {
			opacity: 0;
			display: inline;
		}
		50% {
			opacity: 1;
			display: inline;
		}
		99% {
			opacity: 0;
			display: inline;
		}
		100% {
			display: none;
		}
	}

	.file {
		margin-top: 2em;
	}

	.fileContainer {
		overflow: hidden;
		background: #ebebeb;
		border-radius: 2em;
		padding: 1em 2em;
		color: #212121;
		cursor: pointer;
	}

	.fileContainer [type='file'] {
		display: none;
	}

	#player {
		height: 100%;
		width: 100%;
	}
</style>
