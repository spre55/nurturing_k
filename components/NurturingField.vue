<template>
  <v-row class="ma-10" justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-row justify="center" align="center">
        <div class="love-gage">
          <meter max="100" high="90" low="50" optimum="0" min="0" :value="characterLove"></meter>
        </div>
      </v-row>
      <v-row justify="center" align="center">
        <canvas ref="canvas" width="300" height="500"></canvas>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'NurturingField',
  props: {
    monsterID: {
      type: String,
      required: true,
    },
    love: {
      type: Number,
      default: 0,
      required: true,
    },
  },
  data: function () {
    return {
      // キャラクターの情報
      character: null, 
      characterLove: 0,
      // 移動パラメータ
      state: 'usually',
      timestamp: Date.now(),
      directionX: 1,
      directionY: 1,
      buffer: 0,
      // 画像座標パラメータ
      x: 100,
      y: 250,
      col: 256,
      row: 256, 
      step: 0,
      // 長押し
      timerId: null,
    }
  },
  watch: {
    characterLove: function (n, o) {
      if (n >= 100) {
        this.state = 'death'
      }
    },
  },
  created() {
    this.character = new Image();
    this.character.src = "/monster/" + this.monsterID + "/character.png";
    this.characterLove = this.love
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d");
    this.drawing();
    this.canvas.addEventListener('click', this.onClick, false);
    this.canvas.addEventListener('mousedown', this.onClickStart, false);
    this.canvas.addEventListener('mouseup', this.onClickEnd, false);
  },
  beforeDestroy() {
    this.canvas.removeEventListener('click', this.onClick, false);
    this.canvas.removeEventListener('mousedown', this.onClickStart, false);
    this.canvas.removeEventListener('mouseup', this.onClickEnd, false);
  },
  methods: {
    drawing() {
      // メインの描画処理
      let draw = () => {
        if (Date.now() < (this.timestamp + this.buffer)) return requestAnimationFrame(draw);
        this.context.clearRect(0, 0, 300, 500); // 画面リフレッシュ
        switch (this.state) {
          case 'eating':
            this.eating();
            break;
          case 'stroking':
            this.stroking();
            break;
          case 'death':
            this.death();
            break;
          case 'ascension':
            this.ascension();
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
      this.context.drawImage(this.character, 0, this.col * this.step, this.row, this.col, this.x, this.y, 100, 100);
      this.step++;
      if (this.step > 3) {
        this.step = 0;
      }

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
        if (Math.floor(Math.random() * 10) === 0) {
          this.directionX *= -1;
        }
        if (Math.floor(Math.random() * 10) === 0) {
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
      this.context.drawImage(this.character, 0, this.col * 4, this.row, this.col, this.x, this.y, 100, 100);
      this.state = 'usually';
      this.buffer = 1000;
      this.characterLove+=10;
    },
    stroking() {
      this.context.drawImage(this.character, 0, this.col * 5, this.row, this.col, this.x, this.y, 100, 100);
      this.state = 'usually';
      this.buffer = 1000;
      this.characterLove+=15;
    },
    death() {
      this.context.drawImage(this.character, 0, this.col * 6, this.row, this.col, this.x, this.y, 100, 100);
      this.state = 'ascension';
      this.buffer = 1000;
    },
    ascension() {
      this.context.drawImage(this.character, 0, this.col * 7, this.row, this.col, this.x, this.y, 100, 100);
      this.y--;
      this.buffer = 0;
      if (this.y < -80) {
        this.state = 'usually'
        this.x = 100;
        this.y = 250;
        this.directionX = 1;
        this.directionY = 1;
        this.characterLove = 0;
        this.buffer = 300;
      } 
    },
    onClick(e) {
      let rect = e.target.getBoundingClientRect();
      let x = e.clientX - rect.left;
      let y = e.clientY - rect.top;

      if (this.x <= x && x <= this.x + 100 && this.y <= y && y <= this.y + 100) {
        if (this.state === 'usually') {
          this.state = 'stroking'
        }
      }
    },
    onClickStart(e) {
      let rect = e.target.getBoundingClientRect();
      let x = e.clientX - rect.left;
      let y = e.clientY - rect.top;

      if ( x < this.x || this.x + 100 < x || y < this.y || this.y + 100 < y) {
        if (this.state === 'usually') {
          this.timerId = setInterval(()=>{
            
            if (this.x + 25 <= x && x <= this.x + 75 && this.y + 25 <= y && y <= this.y + 75) {
              if (this.state === 'usually') {
                this.state = 'eating'
              }
            } else {
              // 押した座標に向かって来る
              if (x < this.x + 25) {
                this.directionX = -1;
              } else {
                this.directionX = 1;
              }

              if (y < this.y + 25) {
                this.directionY = -1;
              } else {
                this.directionY = 1;
              }

              this.x += (this.directionX * Math.min(10, Math.abs(this.x + 25 - x)));
              this.y += (this.directionY * Math.min(10, Math.abs(this.y + 25 - y)));

            }

          }, 200);
          
        }
      }
    },
    onClickEnd(e) {
      clearInterval(this.timerId);
    },
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