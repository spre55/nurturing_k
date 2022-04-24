<template>
  <v-row class="ma-10" justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-row justify="center" align="center">
        <div class="love-gage">
          <meter max="100" high="90" low="50" optimum="0" min="0" :value="love"></meter>
        </div>
      </v-row>
      <v-row justify="center" align="center">
        <canvas ref="canvas" width="300" height="500"></canvas>
      </v-row>
      <v-row justify="center" align="center">
        <v-btn class="ma-4" v-on:click="changeStateToEating()">エサやり</v-btn>
        <v-btn class="ma-4" v-on:click="changeStateToStroking()">ナデナデ</v-btn>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'NurturingField',
  data: function () {
    return {
      love: 0,
      state: 'usually',
      timestamp: Date.now(),
      directionX: 1,
      directionY: 1,
      x: 100,
      y: 250,
      image: null,
      eatingImage: null,
      bokobokoImage: null,
      strokingImage: null,
      shotenImage: null,
      shoten2Image: null,
      buffer: 0,
    }
  },
  watch: {
    love: function (n, o) {
      if (n >= 100) {
        this.state = 'death'
      }
    },
  },
  created() {
    this.image = new Image();
    this.image.src = '/monster/010/main.png';

    this.eatingImage = new Image();
    this.eatingImage.src = '/monster/010/eating.png';

    this.bokobokoImage = new Image();
    this.bokobokoImage.src = '/monster/010/bokoboko.png';

    this.strokingImage = new Image();
    this.strokingImage.src = '/monster/010/nadenade.png';

    this.shotenImage = new Image();
    this.shotenImage.src = '/monster/010/shoten.png';

    this.shoten2Image = new Image();
    this.shoten2Image.src = '/monster/010/shoten2.png';
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d");
    this.drawing();
  },
  methods: {
    drawing() {
      let draw = () => {
        if (Date.now() < (this.timestamp + this.buffer)) return requestAnimationFrame(draw);
        this.context.clearRect(0, 0, 300, 500); // 画面リフレッシュ
        switch (this.state) {
          case 'eating':
            this.eating();
            break;
          case 'bokoboko':
            this.bokoboko();
            break;
          case 'stroking':
            this.stroking();
            break;
          case 'death':
            this.death();
            break;
          case 'shoten':
            this.shoten();
            break;
          default:
            this.walk();
            break;
        }
        this.timestamp = Date.now();
        requestAnimationFrame(draw);
      }
      draw();
    },
    walk() {
      this.context.drawImage(this.image, this.x, this.y, 100, 100);

      let isNotOver = true
      // 右横にめり込み時
      if (this.canvas.width < this.x + 100) {
        isNotOver = false
        this.directionX = -1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionY *= -1;
        }
      }
      // 左横にめり込み時
      if (this.x <= 0) {
        isNotOver = false
        this.directionX = 1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionY *= -1;
        }
      }

      // 上にめり込み時
      if (this.y <= 0) {
        isNotOver = false
        this.directionY = 1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionX *= -1;
        }
      }
      // 下にめり込み時
      if (this.canvas.height < this.y + 100) {
        isNotOver = false
        this.directionY = -1;
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionX *= -1;
        }
      }
      // 突然の方向転換
      if (isNotOver) {
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionX *= -1;
        }
        if (Math.floor(Math.random() * 2) === 0) {
          this.directionY *= -1;
        }
        this.buffer = 500;
      } else {
        this.buffer = 0;
      }
      this.x += (this.directionX * Math.floor(Math.random() * 5));
      this.y += (this.directionY * Math.floor(Math.random() * 5));
    },
    eating() {
      this.context.drawImage(this.eatingImage, this.x, this.y, 100, 100);
      this.state = 'usually';
      this.buffer = 1000;
      this.love+=10;
    },
    bokoboko() {
      this.context.drawImage(this.bokobokoImage, this.x, this.y, 100, 100);
      this.state = 'usually';
      this.buffer = 1000;
      this.love+=20;
    },
    stroking() {
      this.context.drawImage(this.strokingImage, this.x, this.y, 100, 100);
      this.state = 'usually';
      this.buffer = 1000;
      this.love+=15;
    },
    death() {
      this.context.drawImage(this.shotenImage, this.x, this.y, 100, 100);
      this.state = 'shoten';
      this.buffer = 1000;
    },
    shoten() {
      this.context.drawImage(this.shoten2Image, this.x, this.y, 100, 100);
      this.y--;
      this.buffer = 0;
      if (this.y < -80) {
        this.state = 'usually'
        this.x = 100;
        this.y = 250;
        this.directionX = 1;
        this.directionY = 1;
        this.love = 0;
        this.buffer = 300;
      } 
    },
    changeStateToStroking() {
      if (this.state === 'usually') {
        this.state = 'stroking'
      }
    },
    changeStateToEating() {
      if (this.state === 'usually') {
        this.state = 'eating'
      }
    }
  }
}
</script>

<style scoped>
canvas {
  border: 1px solid #000;
  background: url('/bg_normal.png');
}

meter {
  width: 100%;
}

.love-gage {
  background: url('/heart.png');
  background-size: contain;
  height: 50px;
  width: 300px;
  text-align: center;
  margin: 10px auto;
  font-size: 15pt;
  background-repeat: no-repeat;
  padding-top: 10px;
  background-position-x: center;
}
</style>