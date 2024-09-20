<template>
  <!-- <div class="countdown-container">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <button @click="downloadImage">Download Image</button>
  </div> -->
  <div class="countdown-container">
    <!-- Elemen video -->
    <video ref="video" :src="videoUrl" @play="drawOnVideo" @loadedmetadata="setCanvasSize" controls></video>
    <!-- Canvas untuk overlay teks -->
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <button @click="downloadVideo">Download Video</button>
  </div>
</template>

<script>
import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg';

export default {
  data() {
    return {
    //   imageUrl: require('@/assets/maulid-countdown.jpg'), // ganti dengan path ke gambar kamu
    //   canvasWidth: 500, // Sesuaikan dengan ukuran gambar
    //   canvasHeight: 300, // Sesuaikan dengan ukuran gambar
    //   countdownDays: 0,

      videoUrl: require('@/assets/background-video.mp4'), // Path video kamu
      canvasWidth: 720,  // Rasio 9:16
      canvasHeight: 1280, // Rasio 9:16
      countdownDays: 0,
      countdownInterval: null,
      ffmpeg: null,
    };
  },
  mounted() {
    this.updateCountdown();
    // this.renderImage();
    setInterval(this.updateCountdown, 1000 * 60 * 60 * 24); // Update hitung mundur setiap 24 jam
  },
  methods: {
    updateCountdown() {
      const targetDate = new Date('2024-11-23T00:00:00'); // Tanggal akhir H-1
      const now = new Date();
      const timeDiff = targetDate - now;
      this.countdownDays = Math.ceil(timeDiff / (1000 * 60 * 60 * 24));
    //   this.renderImage();
    },
    // Menyesuaikan ukuran canvas berdasarkan ukuran video
    setCanvasSize() {
      const video = this.$refs.video;
      const videoRatio = video.videoWidth / video.videoHeight;

      if (videoRatio > 9 / 16) {
        this.canvasHeight = video.videoHeight;
        this.canvasWidth = this.canvasHeight * (9 / 16);
      } else {
        this.canvasWidth = video.videoWidth;
        this.canvasHeight = this.canvasWidth * (16 / 9);
      }

      this.renderText(); // Render teks hitung mundur setelah ukuran canvas diatur
    },
    // Render teks di atas video menggunakan canvas
    drawOnVideo() {
      if (this.countdownInterval) clearInterval(this.countdownInterval);
      this.countdownInterval = setInterval(this.renderText, 1000 / 30); // Render 30 frame per detik
    },
    // Menambahkan teks hitung mundur di atas video
    renderText() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      const video = this.$refs.video;

      // Bersihkan canvas
      ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);

      // Copy frame dari video (opsional, jika butuh frame sinkronisasi)
      ctx.drawImage(video, 0, 0, this.canvasWidth, this.canvasHeight);

      // Tambahkan teks hitung mundur
      ctx.font = 'bold 60px Roboto'; // Menggunakan font kustom Roboto
      ctx.fillStyle = 'white';
      ctx.textAlign = 'center';
      ctx.fillText(`H-${this.countdownDays}`, canvas.width / 2, canvas.height / 2);
    },
    // Proses untuk download video
    async downloadVideo() {
      const ffmpeg = createFFmpeg({ log: true });
      await ffmpeg.load();

      const videoFile = await fetchFile(this.videoUrl);
      ffmpeg.FS('writeFile', 'input.mp4', videoFile);

      const canvas = this.$refs.canvas;
      const imageData = canvas.toDataURL('image/png');
      const data = imageData.split(',')[1];
      const byteString = atob(data);
      const ab = new ArrayBuffer(byteString.length);
      const ia = new Uint8Array(ab);

      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }

      const blob = new Blob([ab], { type: 'image/png' });
      ffmpeg.FS('writeFile', 'overlay.png', await fetchFile(URL.createObjectURL(blob)));

      // Menggunakan filter overlay dari ffmpeg untuk menimpa teks ke video
      await ffmpeg.run(
        '-i',
        'input.mp4',
        '-i',
        'overlay.png',
        '-filter_complex',
        '[0:v][1:v] overlay=W-w-10:H-h-10',
        'output.mp4'
      );

      const dataOutput = ffmpeg.FS('readFile', 'output.mp4');
      const url = URL.createObjectURL(
        new Blob([dataOutput.buffer], { type: 'video/mp4' })
      );

      const a = document.createElement('a');
      a.href = url;
      a.download = 'output-video.mp4';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
    renderImage() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');

      const image = new Image();
      image.src = this.imageUrl;

      image.onload = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(image, 0, 0, this.canvasWidth, this.canvasHeight);

        // Menambahkan teks hitung mundur di atas gambar
        ctx.font = 'bold 40px Arial';
        ctx.fillStyle = 'white';
        ctx.textAlign = 'center';
        ctx.fillText(`H-${this.countdownDays}`, canvas.width / 2, canvas.height / 2);
      };
    },
    downloadImage() {
      const canvas = this.$refs.canvas;
      const link = document.createElement('a');
      link.download = 'countdown-image.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    },
  },
};
</script>

<style>
.countdown-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

video {
  display: block;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none; /* Agar klik langsung ke video */
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
}

/* .countdown-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
} */
</style>
