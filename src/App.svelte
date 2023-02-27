<script>
	import Counter from './lib/Counter.svelte';
	import { onMount } from 'svelte';

	let player;
	let qualityLevels;
	let qualityLevelsButton = [];
	let inputFile;
	let loggingTextarea;
	let logMessage = '';
	let currentLogMessage = '';
	let playerSize = 1280;
	let src = 'https://s3.ap-northeast-2.amazonaws.com/tmp.mediaconvert/sample/const/origin.m3u8';

	const onFileChange = (e) => {};

	const handleUpload = (e) => {
		inputFile.click();
	};

	const handleQuality = (selectedQuality, e) => {
		for (let i = 0; i < qualityLevels.length; i++) {
			const quality = qualityLevels[i];
			if (selectedQuality === 'auto') {
				quality.enabled = true;
			} else {
				if (quality.height === parseInt(selectedQuality)) {
					quality.enabled = true;
				} else {
					quality.enabled = false;
				}
			}
		}
	};

	onMount(() => {
		player = videojs('sample-video', {
			html5: {
				hls: {
					smoothQualityChange: true,
				},
			},
			fill: true,
			responsive: true,
			fluid: true,
			debug: true,
		});

		player.ready(() => {
			qualityLevelsButton = [];
			qualityLevels = player.qualityLevels();
			qualityLevels.on('addqualitylevel', (e) => {
				qualityLevelsButton = [...qualityLevelsButton, e.qualityLevel];
			});
			qualityLevels.on('change', (e) => {
				logMessage += `--> Quality level ID: ${qualityLevels[qualityLevels.selectedIndex_].id} \n`;
				setTimeout(() => {
					loggingTextarea.scrollTop = loggingTextarea.scrollHeight;
				}, 20);
			});
		});

		// player.play();
		// player.on('timeupdate', (e) => {
		// 	let history = videojs.log.history();
		// 	history = history[history.length - 1];
		//
		// 	if (
		// 		history[1] === 'DEBUG:' &&
		// 		!!history[4] &&
		// 		currentLogMessage !== history[4]
		// 	) {
		// 		currentLogMessage = history[4];
		// 		logMessage += `--> ${currentLogMessage} \n`;
		// 		setTimeout(() => {
		// 			loggingTextarea.scrollTop = loggingTextarea.scrollHeight;
		// 		}, 20);
		// 	}
		// });
	});
</script>

<header class="position-fixed top-0 start-0 end-0">
	<h2 class="title m-3 text-start">Video Streaming</h2>
</header>
<main class="position-absolute top-0 bottom-0 start-0 end-0 text-center">
	<div class="content m-3">
		<div class="container">
			<div class="input-group mb-3">
				<input type="text" class="form-control" placeholder=".m3u8 file url 을 입력" value={src} />
				<button class="btn btn-outline-secondary" type="button">확인</button>
			</div>
			<div>또는 직접 업로드</div>
			<div class="m-3">
				<input type="file" hidden bind:this={inputFile} on:change={onFileChange} />
				<button type="button" class="btn btn-outline-secondary" on:click={handleUpload}>
					Video File Upload
				</button>
			</div>
			<div class="video-wrapper mx-auto" style="width: {playerSize}px;">
				<video id="sample-video" class="video-js vjs-16-9 sample-video" controls preload="auto">
					<source {src} type="application/x-mpegURL" />
				</video>
			</div>
			<div class="my-3">
				<div class="btn-group" role="group" aria-label="radio toggle button group">
					{#if !!player && !!qualityLevels}
						<input
							type="radio"
							class="btn-radio"
							id="quality-auto"
							autocomplete="off"
							checked
							on:click={handleQuality.bind(null, 'auto')}
						/>
						<label class="btn btn-outline-primary" for="quality-auto">Auto</label>
						{#each qualityLevelsButton as quality, i}
							<input
								type="radio"
								class="btn-radio"
								id="quality-{quality.height}"
								autocomplete="off"
								on:click={handleQuality.bind(null, quality.height)}
							/>
							<label class="btn btn-outline-primary" for="quality-{quality.height}">
								{quality.height}({Math.round(quality.bitrate / 1024)}k)
							</label>
						{/each}
					{/if}
				</div>
			</div>
			<div class="row m-3 w-50 mx-auto">
				<label class="form-label">Player size: {playerSize}x{(playerSize * 9) / 16}</label>
				<input
					type="range"
					class="form-range"
					min="400"
					max="800"
					step="200"
					bind:value={playerSize}
				/>
			</div>
			<div class="row m-3">
				<textarea
					bind:this={loggingTextarea}
					bind:value={logMessage}
					class="log-message overflow-auto"></textarea>
			</div>
		</div>
	</div>
</main>
<!--<footer class="position-fixed bottom-0 start-0 end-0 text-center">-->
<!--	<p class="read-the-docs">-->
<!--		Create with lambda + svelte + bootstrap + sass + vite-->
<!--	</p>-->

<!--</footer>-->
<style lang="scss">
	header {
		z-index: 1;
		background-color: #8da9c4;

		h2 {
			font-weight: 600;
		}
	}

	main {
		z-index: 0;
		background-color: #eef4ed;
		height: 100vh;
		min-height: 120vh;
		margin-bottom: 20px;

		.content {
			position: relative;
			top: 100px;
		}

		.video-wrapper {
			max-width: 800px;
			position: relative;

			/*img.poster {
				width: 100%;
				height: auto;
				position: relative;
				z-index: 10;
			}
			.video-js {
				height: 100%;
				position: absolute;
				top: 0;
				left: 0;
				width: 100%;
				z-index: 110;
			}*/
		}

		.log-message {
			max-width: 800px;
			min-height: 120px;
			margin: auto;
		}

		.sample-video {
		}
	}

	footer {
		z-index: 1;
	}

	.read-the-docs {
		color: #888;
	}
</style>
