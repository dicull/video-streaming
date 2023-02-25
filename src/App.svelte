<script>
	import Counter from './lib/Counter.svelte';

	let inputFile;

	const onFileChange = (e) => {
		
	};

	const handleUpload = (e) => {
		inputFile.click();
	};
</script>

<header class="position-fixed top-0 start-0 end-0">
	<h2 class="title m-3 text-start">Video Streaming</h2>
</header>
<main class="position-absolute top-0 bottom-0 start-0 end-0 text-center">
	<div class="content m-3">
		<div class="mb-3">
			<input
				type="file"
				hidden
				bind:this="{inputFile}"
				on:change="{onFileChange}" />
			<button
				type="button"
				class="btn btn-outline-secondary"
				on:click="{handleUpload}">Video File Upload</button>
		</div>
		<div>또는</div>
		<div class="input-group mb-3">
			<input
				type="text"
				class="form-control"
				placeholder=".m3u8 file url 을 입력" />
			<button class="btn btn-outline-secondary" type="button"
				>확인</button>
		</div>
		<Counter />
	</div>
</main>
<footer class="position-fixed bottom-0 start-0 end-0 text-center">
	<p class="read-the-docs">
		Create with lambda + svelte + bootstrap + sass + vite
	</p>
</footer>

<style lang="scss">
	header {
		z-index: 1;
		background-color: #8da9c4;
	}

	main {
		z-index: 0;
		background-color: #eef4ed;
		height: 100vh;

		.content {
			position: relative;
			top: 100px;
		}
	}

	footer {
		z-index: 1;
	}

	.read-the-docs {
		color: #888;
	}
</style>
