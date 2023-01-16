<script>
	import { onMount } from "svelte";

	let imageCapture;
	let canvas;

	onMount(async () => {
		if (!("ImageCapture" in window)) {
			console.log("ImageCapture is not supported by your browser");
			console.log('loading polyfill ...')
			await import('../polyfills/imagecapture.js')
			console.log('polyfill loaded')
		}
	})

	async function getUserMedia() {
		try {
			const mediaStream = await navigator.mediaDevices.getUserMedia({ video: true })
			document.querySelector('video').srcObject = mediaStream;
			const track = mediaStream.getVideoTracks()[0];
			imageCapture = new ImageCapture(track);
		} catch(error) {
			console.error(error);
		}
	}

	async function onGrabFrameButtonClick() {
		if (!imageCapture) await getUserMedia();
		imageCapture
			.grabFrame()
			.then((imageBitmap) => {
				drawCanvas(imageBitmap);
			})
			.catch((error) => console.error(error));
	}

	async function onTakePhotoButtonClick() {
		try {
			if (!imageCapture) await getUserMedia();
			const blob = await imageCapture.takePhoto()
			const imageBitmap = await createImageBitmap(blob)
			drawCanvas(imageBitmap);
		} catch(error) {
			console.error(error)
		}
	}

	/* Utils */

	function drawCanvas(img) {
		canvas.width = getComputedStyle(canvas).width.split('px')[0];
		canvas.height = getComputedStyle(canvas).height.split('px')[0];
		let ratio = Math.min(canvas.width / img.width, canvas.height / img.height);
		let x = (canvas.width - img.width * ratio) / 2;
		let y = (canvas.height - img.height * ratio) / 2;
		canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
		canvas
			.getContext('2d')
			.drawImage(img, 0, 0, img.width, img.height, x, y, img.width * ratio, img.height * ratio);
	}

	// document.querySelector('video').addEventListener('play', () => {
	// 	document.querySelector('#grabFrameButton').disabled = false;
	// 	document.querySelector('#takePhotoButton').disabled = false;
	// });
</script>



<button on:click={onTakePhotoButtonClick}>Take Photo</button>
<button on:click={onGrabFrameButtonClick}>Grab Frame</button>
<canvas bind:this={canvas}></canvas>
<video width="200" height="200"></video>