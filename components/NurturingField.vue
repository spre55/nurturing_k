<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <canvas ref="canvas" width="300" height="500"></canvas>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'NurturingField',
  state: 'usually',
  timestamp: Date.now(),
  directionX: 1,
  directionY: 1,
  x: 0,
  y: 0,
  image: null,
  beforeCreate() {
    this.image = new Image();
    this.image.src = '/monster.png';
    this.x = 100;
    this.y = 250;
    this.directionX = 1;
    this.directionY = 1;
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d");
    this.drawing();
  },
  methods: {
    drawing() {
      let draw = () => {
        if (Date.now() < (this.timestamp)) return requestAnimationFrame(draw);
        this.context.clearRect(0,0,300,500);
        this.context.drawImage(this.image, this.x, this.y, 100, 100);
        this.timestamp = Date.now();
        this.walk();
        requestAnimationFrame(draw);
      }
      draw();
    },
    walk() {
      if (this.canvas.width < this.x + 100 || this.x <= 0) {
        this.directionX *= -1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionY *= -1;
        }
      }

      if (this.canvas.height < this.y + 100 || this.y <= 0) {
        this.directionY *= -1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionX *= -1;
        }
      }

      this.x += (this.directionX * Math.floor(Math.random() * 5));
      this.y += (this.directionY * Math.floor(Math.random() * 5));
    },
    changeDirection() {

    },
  },
}
</script>

<style scoped>
canvas {
  border: 1px solid #000;
}
</style>