<script>
	import { onMount } from 'svelte';
	import AWS from 'aws-sdk';

	let player;
	let qualityLevels;
	let qualityLevelsButton = [];
	let inputFile;
	let loggingTextarea;
	let logMessage = '';
	let currentLogMessage = '';
	let playerSize = 960;
	const s3Url = 'https://s3.ap-northeast-2.amazonaws.com';
	const bucket = 'tmp.mediaconvert';
	const sampleKey = 'sample/const/origin.m3u8';
	let src = `${s3Url}/${bucket}/${sampleKey}`;
	let isLoading = false;

	const onFileChange = (e) => {
		upload();
	};

	const handleUpload = (e) => {
		inputFile.click();
	};

	const handleQuality = (selectedQuality, e) => {
		for (let i = 0; i < qualityLevels.length; i++) {
			const quality = qualityLevels[i];
			if (selectedQuality === 'auto') {
				quality.enabled = true;
			} else {
				quality.enabled = quality.height === parseInt(selectedQuality);
			}
		}
	};

	const upload = () => {
		const { VITE_AWS_REGION, VITE_AWS_ACCESS, VITE_AWS_SECRET } = import.meta.env;
		const s3 = new AWS.S3({
			accessKeyId: VITE_AWS_ACCESS,
			secretAccessKey: VITE_AWS_SECRET,
			region: VITE_AWS_REGION,
		});

		const file = inputFile.files[0];
		console.log(file);
		if (file.size >= 100 * 1024 * 1024) {
			return alert('100MB less only possible');
		}

		if (!!file && file.type.indexOf('video') >= 0) {
			const key = `upload/${file.name}`;
			const params = {
				Bucket: bucket,
				Key: key,
				ContentType: file.type,
				Body: file,
			};

			isLoading = true;
			s3.putObject(params, function (err, data) {
				if (err) {
					console.error(err);
					alert(err.message);
					isLoading = false;
					return false;
				}
				// 업로드 성공
				console.log(data);
				isLoading = false;
			});
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

<header>
	<img class="position-absolute top-0 start-0 bottom-0" src="/logo_ani.gif" />
	<h2 class="title my-0 mx-3 py-3 text-start">Video Streaming & Transcoding</h2>
</header>
<main class="text-center">
	<div class="content pt-4">
		<div class="container">
			<div class="input-group">
				<input type="text" class="form-control" placeholder=".m3u8 file url" value={src} />
<!--				<button class="btn btn-outline-secondary" type="button">load</button>-->
			</div>
			<div class="m-1">or</div>
			<div class="mb-3">
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
							name="radio"
							class="btn-check"
							id="quality-auto"
							autocomplete="off"
							checked
							on:click={handleQuality.bind(null, 'auto')}
						/>
						<label class="btn btn-outline-primary" for="quality-auto">Auto</label>
						{#each qualityLevelsButton as quality, i}
							<input
								type="radio"
								name="radio"
								class="btn-check"
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
					min="320"
					max="960"
					step="320"
					bind:value={playerSize}
				/>
			</div>
			<div class="row m-3">
				<textarea
					bind:this={loggingTextarea}
					bind:value={logMessage}
					class="log-message overflow-auto"
				/>
			</div>
		</div>
	</div>
</main>
<footer class="text-center">
	<p class="read-the-docs">Create with svelte + bootstrap + vite + lambda + mediaconvert</p>
</footer>
<div class="loading position-fixed top-0 bottom-0 start-0 end-0" class:show-loading={isLoading}>
	<div class="loading-img-wrapper">
		<img src="/loading.gif" />
	</div>
</div>

<style lang="scss">
	header {
		background-color: #8da9c4;

		h2 {
			font-weight: 600;

			&.title {
				margin-left: 80px !important;
			}
		}

		img {
			//width: 70px;
			height: 60px;
			margin-left: 10px;
			margin-top: 5px;
		}
	}

	main {
		z-index: 0;
		background-color: #eef4ed;
		min-height: 100vh;
		margin-bottom: 20px;

		.video-wrapper {
			max-width: 960px;
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
	}

	.read-the-docs {
		color: #888;
	}

	.loading {
		display: none;
		background-color: rgba(0, 0, 0, 0.7);

		&.show-loading {
			display: block;
		}

		.loading-img-wrapper {
			position: absolute;
			top: 50%;
			right: 50%;
			transform: translate(50%, -50%);
		}

		img {
			width: 50%;
			transform: rotate(20deg);
		}
	}
</style>
