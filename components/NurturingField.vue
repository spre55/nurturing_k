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
  x: 0,
  y: 0,
  image: null,
  beforeCreate() {
    this.image = new Image();
    this.image.src = '/monster.png';
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.drawing(this.canvas.getContext("2d"))
  },
  methods: {
    drawing(context) {
      let draw = () => {
        if (Date.now() < (this.timestamp + 1800)) return window.requestAnimationFrame(draw);
        context.drawImage(this.image, this.x, this.y, 100, 100);
        this.timestamp = Date.now();
        window.requestAnimationFrame(draw);
      }
      draw();
    }
  },
}
</script>

<style scoped>
canvas {
  border: 1px solid #fff;
}
</style>