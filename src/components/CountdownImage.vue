<template>
  <div class="countdown-container">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <button @click="downloadImage">Download Image</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imageUrl: '/images/maulid-nabi.jpg', // ganti dengan path ke gambar kamu
      canvasWidth: 320, // Sesuaikan dengan ukuran gambar
      canvasHeight: 600, // Sesuaikan dengan ukuran gambar
      countdownDays: 0,
    };
  },
  mounted() {
    this.updateCountdown();
    this.renderImage();
    setInterval(this.updateCountdown, 1000 * 60 * 60 * 24); // Update hitung mundur setiap 24 jam
  },
  methods: {
    updateCountdown() {
      const targetDate = new Date('2024-11-23T00:00:00'); // Tanggal akhir H-1
      const now = new Date();
      const timeDiff = targetDate - now;
      this.countdownDays = Math.ceil(timeDiff / (1000 * 60 * 60 * 24));
      this.renderImage();
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
        ctx.font = 'bold 65px Arial';
        ctx.fillStyle = 'black';
        ctx.textAlign = 'center';
        ctx.fillText(`H-${this.countdownDays}`, (canvas.width / 2)+0, (canvas.height / 2)-188);
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
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>