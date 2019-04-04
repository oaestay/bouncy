<template>
  <div class="app">
    <div class="app__canvas-container" id="canvas-container" ref="canvasContainer"></div>
  </div>
</template>

<script>
import _ from 'lodash';
import p5 from 'p5';

const DISTANCE_CORRECTION = 10;
const DEVISE_ROTATION_CORRECTION = 0.5;
const DEVISE_ACCELERATION_CORRECTION = 0.5;
const ENERGY_LOSS_CONSTANT = 0.9;

export default {
  name: 'app',
  data() {
    return {
      p5Instance: null,
      balls: [],
    };
  },
  methods: {
    draw() {
      this.p5.background(0);

      this.balls.forEach((ball) => {
        this.p5.push();
        this.p5.fill(ball.color);
        this.p5.noStroke();
        this.p5.circle(ball.x, ball.y, ball.radius);
        this.p5.pop();
      });
      this.transitionBalls();
    },
    mouseClicked() {
      this.createBall();
    },
    setup() {
      const canvas = this.p5.createCanvas(this.p5.windowWidth, this.p5.windowHeight);
      canvas.parent('canvas-container');
    },
    windowResized() {
      this.p5.resizeCanvas(this.p5.windowWidth, this.p5.windowHeight);
    },
    intializeP5() {
      if (this.p5) return;
      this.p5 = new p5((p) => {
        this.p5 = p;
        this.p5.draw = this.draw;
        this.p5.mouseClicked = this.mouseClicked;
        this.p5.setup = this.setup;
        this.p5.windowResized = this.windowResized;
      }, this.$refs.p5);
    },
    createBall() {
      this.balls = [
        ...this.balls.map(ball => ({ ...ball })),
        {
          x: this.p5.mouseX,
          y: this.p5.mouseY,
          vX: 0,
          vY: 0,
          aX: 0,
          aY: 9.8,
          radius: _.random(20, 50),
          color: this.p5.color(_.random(255), _.random(255), _.random(255)),
        },
      ];
    },
    transitionBalls() {
      const frameRate = this.p5.getFrameRate();
      const {
        height, width, rotationX, rotationY, accelerationX, accelerationY,
      } = this.p5;
      this.balls = this.balls.map((ball) => {
        let x = ball.x + ball.vX / frameRate * DISTANCE_CORRECTION;
        let y = ball.y + ball.vY / frameRate * DISTANCE_CORRECTION;
        let vX = ball.vX + (
          ball.aX + rotationY * DEVISE_ROTATION_CORRECTION
            + accelerationX * DEVISE_ACCELERATION_CORRECTION
        ) / frameRate * DISTANCE_CORRECTION;
        let vY = ball.vY + (
          ball.aY + rotationX * DEVISE_ROTATION_CORRECTION
            + accelerationY * DEVISE_ACCELERATION_CORRECTION
        ) / frameRate * DISTANCE_CORRECTION;
        if (x < 0) {
          x = -x;
          vX = -vX * ENERGY_LOSS_CONSTANT;
        } else if (width < x) {
          x = 2 * width - x;
          vX = -vX * ENERGY_LOSS_CONSTANT;
        }
        if (y < 0) {
          y = -y;
          vY = -vY * ENERGY_LOSS_CONSTANT;
        } else if (height < y) {
          y = 2 * height - y;
          vY = -vY * ENERGY_LOSS_CONSTANT;
        }
        return {
          ...ball, x, y, vX, vY,
        };
      });
    },
  },
  created() {
    this.intializeP5();
  },
  destroyed() {
    this.p5.remove();
  },
};
</script>

<style lang="scss">
.app {
  height: 100vh;
  margin: 0;
  padding: 0;
  width: 100vw;
  overflow: hidden;

  &__canvas-container {
    height: 100vh;
    margin: 0;
    padding: 0;
    width: 100vw;
  }
}
</style>
