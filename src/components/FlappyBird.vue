<template>

  <div id="wrap" v-loading="loading" element-loading-text="Loading...">

    <div>
      <div id="sky" ref="sky"></div>
      <div id="bird" ref="bird"></div>
    </div>

    <section class="pipes-wrap" ref="pipes">
      <div class="pipe-item" v-for="(item, index) in pipeArr" :key="item.id" :id="'pipe' + index"
        :style="'right:' + item.right + 'px;'">
        <div class="pipe pipe-top" :style="'top:-' + item.top + 'px;'"></div>
        <div class="pipe pipe-bottom" :style="'bottom:-' + item.bottom + 'px;'"></div>
      </div>
    </section>

    <div id="score" ref="score">
      Score: {{ score }}
    </div>

  </div>

</template>

<script>

import { ElMessageBox } from 'element-plus';
import { FaceMeshDetection, DIRECTION } from './FaceMeshDetection.js';

const START_SPEED = 1;
const KEY = 'Space';
const SPEED = 0.005;                  // 加速度
const UP = 5.0;                       // 速度累加上限
const UP_SUM = 100;                   // 按一次跳跃的高度
const BIRD_WIDTH = 50;                // 小鸟图片宽50px
const PIPE_DISTANCE = 350;            // 管道之间的横向距离
const SPACE_HEIGHT = [360, 300, 240]; // 上管道与下管道之间的距离
const x = 300;
const y = 300;

export default {

  name: 'FlappyBird',

  data() {
    return {
      loading: true,
      clientWidth: 0,
      clientHeight: 0,
      id: 0,                        // 管道唯一id, 从0开始计数
      pipeArr: [],                  // 管道数组
      score: 0,                     // 得分
      speed: START_SPEED,           // 初始速度
      jumpSum: 0,                   // 当前跳跃相对高度
      jumpFlag: false,              // true: 按下空格键跳跃上升阶段; false: 自由落体阶段
      gameOver: false,              // 游戏失败的flag, 用于停止动画帧
    };
  },

  mounted() {
    this.clientHeight = document.documentElement.clientHeight;
    this.clientWidth = document.documentElement.clientWidth;

    // 设置按键绑定
    document.onkeydown = event => {
      this.handleKeyPress(event.code);
    };

    // 设置头部动作绑定
    let facemeshDet = new FaceMeshDetection(dir => {
      if (this.loading) { // 加载完成
        this.loading = false;
      }
      else if (dir == DIRECTION.DOWN) {
        console.log('down');
        this.handleKeyPress(KEY);
      }
    });
    facemeshDet.debounceTime = 300;
    facemeshDet.init();

    // 设置动画循环
    let animationLoop = () => {
      if (!this.gameOver && !this.loading) {
        this.loop();
        this.pipesMove();
      }
      requestAnimationFrame(animationLoop);
    }
    animationLoop();

    this.init(); // 初始化数据

  },

  methods: {
    init() {
      // 小鸟位置复原
      let bird = this.$refs.bird;
      bird.style.top = `${y}px`;
      bird.style.right = `${x}px`;

      // 数据复原
      this.score = 0;
      this.id = 0;
      this.speed = START_SPEED;
      this.pipeArr = [];
      this.addPipe(this.id);
      this.gameOver = false;
    },

    loop() {
      //监测跳跃flag
      if (this.jumpFlag == true) {
        this.jump();
      }
      else {
        let top = bird.offsetTop;
        //碰到边界判断游戏结束
        if (top > this.clientHeight - BIRD_WIDTH || top <= 0) {
          this.resetGame();
          return;
        }
        //小鸟降落
        bird.style.top = top + this.speed * this.speed + "px";
        //给予小鸟加速度
        if (this.speed < UP) {
          this.speed += SPEED;
        }
      }
    },

    jump() {
      if (this.jumpSum > UP_SUM) {
        // 到顶部就落下
        this.jumpFlag = false;
        this.jumpSum = 0;
        this.speed = START_SPEED;
      } else {
        bird.style.top = bird.offsetTop - 8 + "px";
        this.jumpSum += 8;
      }
    },
    handleKeyPress(key) {
      if (key == KEY) {
        this.jumpFlag = true;
      }
    },

    // 添加柱子
    addPipe(id) {
      let top_num = this.sum(10, 170);
      let height = SPACE_HEIGHT[ // 随机选取间隙值
        Math.floor(Math.random() * SPACE_HEIGHT.length)
      ];
      let bottom_num = height - top_num;
      let obj = {
        top: height - top_num,
        id: id,
        right: -(PIPE_DISTANCE / 2),
        bottom: bottom_num
      };
      this.pipeArr.push(obj);
    },

    // 随机n-m之间的数字
    sum(m, n) {
      return Math.floor(Math.random() * (m - n) + n);
    },

    // 柱子移动
    pipesMove() {
      //管道列表为空，退出
      if (this.pipeArr.length === 0) {
        return;
      }
      //管道列表中的首位已经越过边界
      let right0 = this.pipeArr[0].right;
      if (right0 > this.clientWidth + 300) {
        this.pipeArr.shift();
      }
      //当空间剩余时，添加管道
      let right_last = this.pipeArr[this.pipeArr.length - 1].right;
      if (right_last >= PIPE_DISTANCE / 2) {
        this.id++;
        this.addPipe(this.id);
      }
      //管道与小鸟的碰撞监测
      for (let i = 0; i < this.pipeArr.length; i++) {

        // 判断一下小鸟是否接触到了管道，小鸟50*50，left：300px；管道宽100px；管道进入范围right是width-450到width-300
        if (
          this.pipeArr[i].right >= this.clientWidth - 450 &&
          this.pipeArr[i].right <= this.clientWidth - 300
        ) {
          // 该管道进入了小鸟触碰范围
          let bird_top = bird.offsetTop;
          // 12是小鸟图片素材上下有空白间隙
          if (
            bird_top <= this.clientHeight / 2 - this.pipeArr[i].top - 12 ||
            bird_top >= this.clientHeight / 2 + this.pipeArr[i].bottom - BIRD_WIDTH + 12
          ) {
            // 碰到了管道
            this.resetGame();
            return;
          }
        }
        if (this.pipeArr[i].right >= this.clientWidth - 300) { // 当某个管道刚好在小鸟左边，证明小鸟通过该管道，根据管道id算出小鸟得分
          this.score = this.pipeArr[i].id + 1;
        }
        this.pipeArr[i].right = this.pipeArr[i].right + 3; // 管道每帧移动2px
      }
    },

    resetGame() {
      this.gameOver = true;
      ElMessageBox.alert(`Your Score: ${this.score}`, 'Game Over', {
        confirmButtonText: 'Restart',
        showCancelButton: true,
        cancelButtonText: 'Back'
      })
        .then(() => {
          this.init();
        })
        .catch(() => {
          this.$router.push({ name: 'home' });
        })
    }
  }
}

</script>

<style scoped>
#wrap {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

#score {
  color: whitesmoke;
  font-size: 20px;
  position: absolute;
  left: 50px;
  top: 50px;
  padding: 5px 8px;
  border-radius: 5px;
  background-color: rgba(100, 100, 100, 0.7);
}

#bird {
  background-image: url('@/assets/flappybird/bird.png');
  height: 50px;
  width: 50px;
  border-radius: 100%;
  position: absolute;
  left: 300px;
  top: 300px;
}

#sky {
  background: url('@/assets/flappybird/sky.png');
  height: 1000px;
  width: 100%;
  position: absolute;
  left: 0;
  top: 0;
}


#pipes-wrap {
  position: relative;
  height: 100%;
}

.pipe-item {
  position: absolute;
  height: 100%;
  width: 100px;
}

.pipe {
  width: 100%;
  height: 50%;
  position: relative;
}

.pipe-top {
  background: url('@/assets/flappybird/pipe_down.png') no-repeat;
  background-size: 100px;
  background-position: bottom;
}

.pipe-bottom {
  background: url('@/assets/flappybird/pipe_up.png') no-repeat;
  background-size: 100px;
  background-position: top;
}
</style>



