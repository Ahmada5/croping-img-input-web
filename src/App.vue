<template>
	<div>
		<div class="title">
			<p class="d-flex justify-center text-red-lighten-1 font-weight-bold">ID Detection Dashboard</p>
		</div>
		<div>
			<label>Crop Area 1:</label>
			<div>
				<label>Width:</label>
				<input type="number" v-model="cropWidth1">
			</div>
			<div>
				<label>Height:</label>
				<input type="number" v-model="cropHeight1">
			</div>
			<div>
				<label>X Position:</label>
				<input type="number" v-model="cropX1">
			</div>
			<div>
				<label>Y Position:</label>
				<input type="number" v-model="cropY1">
			</div>
		</div>

		<div>
			<label>Crop Area 2:</label>
			<div>
				<label>Width:</label>
				<input type="number" v-model="cropWidth2">
			</div>
			<div>
				<label>Height:</label>
				<input type="number" v-model="cropHeight2">
			</div>
			<div>
				<label>X Position:</label>
				<input type="number" v-model="cropX2">
			</div>
			<div>
				<label>Y Position:</label>
				<input type="number" v-model="cropY2">
			</div>
		</div>
		<div class="flip-horizontal mt-5" style="position: relative; background-color: aqua; overflow: hidden">
			<div style="position: relative">
				<video id="video" autoplay style="width: 100%"></video>
				<div class="crop-area" :style="cropArea1Style" :data-marker="true"></div>
				<div class="crop-area" :style="cropArea2Style" :data-marker="true"></div>
			</div>
		</div>
		<div>
			<div class="mb-5">
				<button id="startbutton" @click="takePicture">Take Picture</button>
				<button class="mr-2 ml-2" id="startbutton" @click="clearPicture">Clear Picture</button>
				<button id="startbutton" @click="sendPicture">Send Picture</button>
			</div>
			<canvas class="marginarea" ref="canvas"></canvas>
			<canvas class="marginarea" ref="croppedCanvas1"></canvas>
			<canvas class="marginarea" ref="croppedCanvas2"></canvas>

		</div>
	</div>
</template>


<script>
import axios from 'axios';

function baseToImage(stringBase, fileName) {
	const parts = stringBase.split(';base64,');
	const fileType = parts[0].split(':')[1];
	const byteCharacters = atob(parts[1]);
	const byteNumbers = new Array(byteCharacters.length);
	for (let i = 0; i < byteCharacters.length; i++) {
		byteNumbers[i] = byteCharacters.charCodeAt(i);
	}
	const byteArray = new Uint8Array(byteNumbers);
	const blob = new Blob([byteArray], { type: fileType });
	const file = new File([blob], fileName, { type: 'jpg' });
	return file;
}

export default {
	data() {
		return {
			width: 620,
			height: 340,
			// Crop area 1
			cropWidth1: 250,
			cropHeight1: 160,
			cropX1: 150,
			cropY1: 160,
			// Crop area 2
			cropWidth2: 250,
			cropHeight2: 100,
			cropX2: 350,
			cropY2: 100,
			finalData1: "",
			finalData2: "",
			globalDate: "",
			globalRandom: "",
		};
	},
	computed: {
		cropArea1Style() {
			return {
				width: `${this.cropWidth1}px`,
				height: `${this.cropHeight1}px`,
				left: `${(this.cropX1 / this.width) * 100}%`,
				top: `${(this.cropY1 / this.height) * 100}%`,
			};
		},
		cropArea2Style() {
			return {
				width: `${this.cropWidth2}px`,
				height: `${this.cropHeight2}px`,
				left: `${(this.cropX2 / this.width) * 100}%`,
				top: `${(this.cropY2 / this.height) * 100}%`,
			};
		},
	},
	mounted() {
		this.startup();
		this.getGlobalComp();
	},
	methods: {
		startup() {
			const video = document.getElementById("video");
			const canvas = this.$refs.canvas;

			const constraints = {
				video: {
					width: { ideal: this.width },
					height: { ideal: this.height },
					facingMode: "user",
				},
				audio: false
			};

			navigator.mediaDevices
				.getUserMedia(constraints)
				.then((stream) => {
					video.srcObject = stream;
					video.play();
				})
				.catch((err) => {
					console.error(`An error occurred: ${err}`);
					alert(`An error occurred while accessing the camera: ${err.message}`);
				});

			video.addEventListener("canplay", () => {
				if (!this.streaming) {
					video.setAttribute("width", this.width);
					video.setAttribute("height", this.height);
					canvas.width = this.width;
					canvas.height = this.height;
					this.streaming = true;
					console.log("Camera streaming started");
				}
			});
		},
		takePicture() {
			console.log(this.globalDate + " " + this.globalRandom);
			const video = document.getElementById("video");
			const canvas = this.$refs.canvas;
			const context = canvas.getContext("2d");
			if (canvas && context && video) {
				// Draw the full video frame on the canvas
				context.drawImage(video, 0, 0, canvas.width, canvas.height);

				const fullImageData = canvas.toDataURL("image/png");
				this.photoUrl = fullImageData;

				// Create a new canvas for cropping area 1
				const cropCanvas1 = this.$refs.croppedCanvas1;
				const cropContext1 = cropCanvas1.getContext('2d');
				cropCanvas1.width = this.cropWidth1;
				cropCanvas1.height = this.cropHeight1;

				// Draw the cropped area 1 on the new canvas
				cropContext1.drawImage(
					canvas,
					this.cropX1, this.cropY1, this.cropWidth1, this.cropHeight1,
					0, 0, this.cropWidth1, this.cropHeight1
				);

				const croppedData1 = cropCanvas1.toDataURL("image/png");
				this.finalData1 = baseToImage(croppedData1, this.globalDate + "_1");
				console.log(this.finalData1);

				// Create a new canvas for cropping area 2
				const cropCanvas2 = this.$refs.croppedCanvas2;
				const cropContext2 = cropCanvas2.getContext('2d');
				cropCanvas2.width = this.cropWidth2;
				cropCanvas2.height = this.cropHeight2;

				// Draw the cropped area 2 on the new canvas
				cropContext2.drawImage(
					canvas,
					this.cropX2, this.cropY2, this.cropWidth2, this.cropHeight2,
					0, 0, this.cropWidth2, this.cropHeight2
				);

				const croppedData2 = cropCanvas2.toDataURL("image/png");
				this.finalData2 = baseToImage(croppedData2, this.globalDate + "_2");
				console.log(this.finalData2);
			}
		},
		clearPicture() {
			const canvas = this.$refs.canvas;
			const cropCanvas1 = this.$refs.croppedCanvas1;
			const cropCanvas2 = this.$refs.croppedCanvas2;
			if (canvas && cropCanvas1 && cropCanvas2) {
				const context = canvas.getContext("2d");
				const cropContext1 = cropCanvas1.getContext("2d");
				const cropContext2 = cropCanvas2.getContext("2d");
				if (context && cropContext1 && cropContext2) {
					context.fillStyle = "azure";
					context.fillRect(0, 0, canvas.width, canvas.height);
					cropContext1.fillStyle = "azure";
					cropContext1.fillRect(0, 0, cropCanvas1.width, cropCanvas1.height);
					cropContext2.fillStyle = "azure";
					cropContext2.fillRect(0, 0, cropCanvas2.width, cropCanvas2.height);
					this.finalData1 = "";
					this.finalData2 = "";
				}
			}
		},
		async getData() {
			try {
				const response = await axios.get('http://localhost:3000/files');
				this.responseData = response.data;
				console.log(this.responseData);
			} catch (error) {
				console.error('Error fetching data', error);
			}
		},
		async sendPicture() {
			if (this.finalData1 && this.finalData2) {
				const formData = new FormData();
				formData.append('id', this.globalRandom);
				formData.append('date', this.globalDate);
				formData.append('image1', this.finalData1);
				formData.append('image2', this.finalData2);
				console.log(formData);
				try {
					const response = await axios.post('http://bayu.com:5000/sim-read', formData, {
						headers: {
							'Content-Type': 'multipart/form-data'
						}
					});
					console.log("Response:", response.data);
					console.log("Upload success!");
				} catch (error) {
					console.error('Error:', error);
				}
			} else {
				console.error('No picture data available');
			}
		},
		getGlobalComp() {
			const globalDate = new Date();
			const randomNumber = Math.floor(Math.random() * 10000);

			const day = ('0' + globalDate.getDate()).slice(-2);
			const month = ('0' + (globalDate.getMonth() + 1)).slice(-2);
			const year = globalDate.getFullYear();

			const finalDate = day + month + year;
			this.globalDate = finalDate;
			this.globalRandom = randomNumber;
		}
	},
	
};
</script>

<style scoped>
#video.flip-horizontal {
	transform: scaleX(-1);
}

#canvas {
	border: 1px solid black;
}

.crop-area {
	border: 2px solid rgba(255, 0, 0, 0.5);
	position: absolute;
	pointer-events: none;
	margin: 0;
}

.fullscreen {
	width: 100vw;
	height: 100vh;
}
</style>