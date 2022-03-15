<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <div>
    <!-- <button @click.prevent="stop">stop</button> -->
    <div class="start" v-if="status == 'start'">
      <div class="d-flex justify-content-center align-items-center h-100">
        <div class="text-center">
          <h1>SNAAAAAAKE</h1>
          <p v-if="media=='pc'">PRESS [<span class="bang">START</span>] TO START</p>
          <p v-if="media=='mobile'">PRESS [<span class="bang" @click="start(32)">HERE</span>] TO START</p>
        </div>
      </div>
    </div>
    <div class="play" v-if="status =='play'">
      <div class="d-flex justify-content-center align-items-center h-100">
        <div class="board">
          <div class="row no-gutters" v-for="i in height" :key="i">
            <div class="box" v-for="j in width" :key="j">
              <div></div>
            </div>
          </div>

          <div class="words lt">
            SNAA
          </div>
          <div class="words tl">
            AAAAKE
          </div>
          <div class="words rb">
            SNAA
          </div>
          <div class="words br">
            AAAAKE
          </div>
          <div class="best">
            best
            <span> {{bestScore}} </span>
          </div>
          <div class="score">
            score
            <span> {{score}} </span>
          </div>
        </div>
      </div>
    </div>
    <div class="end" v-if="status == 'end'">
      <div class="d-flex justify-content-center align-items-center h-100">
        <div class="text-center">
          <h1>GAME OVER</h1>
          <div class="d-flex align-items-center my-3" style="justify-content: space-between;">
            <div>
              score
              <p style="font-size: 36px;"> {{score}} </p>
            </div>
            <div style="font-size: 36px;">
              /
            </div>
            <div>
              best
              <p style="font-size: 36px;"> {{bestScore}} </p>
            </div>
          </div>
          <span class="p-2" style="font-size: 28px; color: #000; background-color: #fff;">RESTART?</span>
          <div class="d-flex align-items-center mt-5"
            style="justify-content: space-between; color: #00FFE2; font-size: 22px;">
            <div @click="again(89)">
              YES[Y]
            </div>
            <div @click="again(78)">
              NO[N]
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  // import HelloWorld from './components/HelloWorld.vue'

  export default {
    name: 'App',
    components: {
      // HelloWorld
    },
    data() {
      return {
        status: 'start',
        media: '',
        width: 28,
        height: 16,
        alive: true,
        direction: [0, 0],
        interval: 0,
        score: 0,
        bestScore: 0,
        alive: true,
        speed: 400,
        positionRecord: [{
          x: 0,
          y: 0
        }],
        pointPos: {
          x: 0,
          y: 0
        }
      }

    },
    created() {
      this.boardSet()

    },
    mounted() {
      this.boardSet()
      window.addEventListener('resize', this.boardSet)

      this.positionRecord[0] = {
        x: Math.round(this.width / 2),
        y: Math.round(this.height / 2)
      }
      document.addEventListener('keydown', this.start)
      if (localStorage.getItem('best') != null) {
        this.bestScore = localStorage.getItem('best')
      }
    },
    watch: {
      score: function () {
        if (this.score >= 60) {
          this.speed = 50
        } else if (this.score >= 40) {
          this.speed = 100
        } else if (this.score >= 20) {
          this.speed = 150
        } else if (this.score >= 10) {
          this.speed = 200
        } else if (this.score >= 5) {
          this.speed = 300
        }
      }
    },
    methods: {
      /**
       * 遊戲桌尺寸設定
       */
      boardSet() {
        let scrollWidth = document.documentElement.scrollWidth
        if (scrollWidth > 992) {
          this.width = 28
          this.height = 16
          this.media = 'pc'
        } else if (scrollWidth > 768) {
          this.width = 28
          this.height = 16
          this.media = 'pc'
        } else if (scrollWidth > 576) {
          this.width = 16
          this.height = 16
          this.media = 'pc'
        } else {
          this.width = 14
          this.height = 20
          this.media = 'mobile'
        }
      },
      /**
       * 按下空白鍵會開始遊戲
       * @param {Event} e 觸發事件
       */
      start(e) {
        let _this = this
        let type = ''
        e.keyCode ? type = e.keyCode : type = e
        if (type == 32) { //按下空白鍵
          _this.status = 'play'
          document.removeEventListener('keydown', _this.start)
          _this.$nextTick(() => {
            document.addEventListener('keydown', _this.keycodeControl)
            document.addEventListener('touchstart', _this.touch, {
              passive: false
            });
            document.addEventListener('touchmove', _this.touch, {
              passive: false
            });
            document.addEventListener('touchend', _this.touch, {
              passive: false
            });
            _this.food()
            _this.drawSnake()
          })
        }
      },
      /**
       * 手機觸控事件
       * @param {Event} e 
       */
      touch(e) {
        let _this = this
        switch (e.type) {
          case "touchstart":
            startX = e.touches[0].pageX;
            startY = e.touches[0].pageY;
            break;
          case "touchend":
            var spanX = e.changedTouches[0].pageX - startX;
            var spanY = e.changedTouches[0].pageY - startY;

            if (Math.abs(spanX) > Math.abs(spanY)) { //水平方向
              if (spanX > 30) { //向右
                _this.keycodeControl('right')
              } else if (spanX < -30) { //向左
                _this.keycodeControl('left')
              }
            } else { //垂直方向
              if (spanY > 30) { //向下
                _this.keycodeControl('down')
              } else if (spanY < -30) { //向上
                _this.keycodeControl('up')
              }
            }

            break;
          case "touchmove":
            e.preventDefault();
            break;
        }
      },
      /**
       * 控制蛇的前進方向
       * @param {Event} 觸發事件 
       */
      keycodeControl(e) {
        let type = ''
        e.keyCode ? type = e.keyCode : type = e
        switch (type) {
          case 37:
          case 'left': //向左
            this.direction[0] != 1 || this.positionRecord.length == 1 ? this.directionControl([-1, 0]) : false
          break
          case 38:
          case 'up': //向上
            this.direction[1] != 1 || this.positionRecord.length == 1 ? this.directionControl([0, -1]) : false
          break
          case 39:
          case 'right': //向右
            this.direction[0] != -1 || this.positionRecord.length == 1 ? this.directionControl([1, 0]) : false
          break
          case 40:
          case 'down': //向下
            this.direction[1] != -1 || this.positionRecord.length == 1 ? this.directionControl([0, 1]) : false
          break
        }
      },
      /**
       * 持續觸發移動function
       * @param {Array} direction 前進的方向
       */
      directionControl(direction) {
        this.status = 'play'
        clearInterval(this.interval) //清除先前的前進方向
        this.direction = direction //紀錄目前方向
        this.move(direction)
        this.interval = setInterval(() => {
          this.move(direction)
        }, this.speed);
      },
      /**
       * 檢查穿牆、走過的地方還原、檢查得分
       * @param {Array} direction 前進方向
       */
      move(direction) {
        let _this = this
        let snakeLength = _this.positionRecord.length
        let headPos = _this.positionRecord[snakeLength - 1]
        let footPos = _this.positionRecord[0]
        let rows = document.querySelectorAll(".row")
        let newPosition = {
          x: (headPos.x + direction[0] + _this.width) % _this.width,
          y: (headPos.y + direction[1] + _this.height) % _this.height
        }
        _this.positionRecord.push(newPosition)
        rows[footPos.y].children[footPos.x].children[0].style = ''
        _this.setFoodShawdow(_this.pointPos.x, _this.pointPos.y)
        if (newPosition.x == _this.pointPos.x && newPosition.y == _this.pointPos.y) { //得分
          _this.score++
          _this.removeFood()
          _this.food()
        } else { //未得分去掉後面
          _this.positionRecord.splice(0, 1)
        }
        _this.drawSnake()
      },
      /**
       * 蛇的身體、陰影、檢查碰撞
       */
      drawSnake() {
        let _this = this
        if (_this.alive) {
          let snakeLength = _this.positionRecord.length
          let headPos = _this.positionRecord[snakeLength - 1]
          let rows = document.querySelectorAll(".row")
          for (let i = 0; i < snakeLength; i++) {
            rows[_this.positionRecord[i].y].children[_this.positionRecord[i].x].children[0].style.backgroundColor = '#00FFE2'
            rows[_this.positionRecord[i].y].children[_this.positionRecord[i].x].children[0].style.opacity = (i + 1) / snakeLength
            rows[_this.positionRecord[i].y].children[_this.positionRecord[i].x].children[0].style['box-shadow'] = 'none'
            rows[headPos.y].children[headPos.x].children[0].style['box-shadow'] = '0 0 30px 6px #fff'
          }
          // 碰撞檢查
          for (let i = 0; i < snakeLength - 1; i++) {
            //Game Over
            if (headPos.x == _this.positionRecord[i].x && headPos.y == _this.positionRecord[i].y) {
              rows[headPos.y].children[headPos.x].children[0].style.backgroundColor = '#ff7600'
              _this.alive = false
              if (_this.score > _this.bestScore) {
                _this.bestScore = _this.score
              }
              localStorage.setItem('best', _this.score)
              setTimeout(() => {
                _this.status = 'end'
                document.removeEventListener('keydown', _this.keycodeControl)
                document.removeEventListener('touchstart', _this.touch, {
                  passive: false
                });
                document.removeEventListener('touchmove', _this.touch, {
                  passive: false
                });
                document.removeEventListener('touchend', _this.touch, {
                  passive: false
                });
                document.addEventListener('keydown', _this.again)
              }, 1500);
            }
          }
        } else {
          clearInterval(_this.interval);
        }
      },
      /**
       * 食物隨機位置
       */
      food() {
        let rows = document.querySelectorAll(".row")
        let randomX = Math.floor(Math.random() * this.width)
        let randomY = Math.floor(Math.random() * this.height)
        let repeat = this.positionRecord.filter(item => randomX == item.x && randomY == item.y)
        if (repeat.length > 0) {
          this.food()
          return false
        }
        this.pointPos = {
          x: randomX,
          y: randomY
        }
        rows[randomY].children[randomX].children[0].setAttribute('class','food-img')
        this.setFoodShawdow(this.pointPos.x, this.pointPos.y)
      },
      /**
       * 設定食物陰影
       * @param {Number} x 食物x軸
       * @param {Number} y 食物y軸
       */
      setFoodShawdow(x, y) {
        let rows = document.querySelectorAll(".row")
        for (let i = 1; i < 8; i++) {
          let opacityValue = (1 - i / 8) * 0.2
          if (x - i >= 0) {
            rows[y].children[x - i].children[0].style.backgroundColor = `rgba(255, 255, 255, ${opacityValue})`
          }
          if (x + i <= (this.width - 1)) {
            rows[y].children[x + i].children[0].style.backgroundColor = `rgba(255, 255, 255, ${opacityValue})`
          }
          if (y - i >= 0) {
            rows[y - i].children[x].children[0].style.backgroundColor = `rgba(255, 255, 255, ${opacityValue})`
          }
          if (y + i <= (this.height - 1)) {
            rows[y + i].children[x].children[0].style.backgroundColor = `rgba(255, 255, 255, ${opacityValue})`
          }
        }
      },
      /**
       * 移除食物
       */
      removeFood() {
        let rows = document.querySelectorAll(".row")
        let _this = this
        for (let i = 1; i < 8; i++) {
          if (_this.pointPos.x - i >= 0) {
            rows[_this.pointPos.y].children[_this.pointPos.x - i].children[0].style.backgroundColor = ``
          }
          if (_this.pointPos.x + i <= (this.width - 1)) {
            rows[_this.pointPos.y].children[_this.pointPos.x + i].children[0].style.backgroundColor = ``
          }
          if (_this.pointPos.y - i >= 0) {
            rows[_this.pointPos.y - i].children[_this.pointPos.x].children[0].style.backgroundColor = ``
          }
          if (_this.pointPos.y + i <= (this.height - 1)) {
            rows[_this.pointPos.y + i].children[_this.pointPos.x].children[0].style.backgroundColor = ``
          }
        }
        rows[_this.pointPos.y].children[_this.pointPos.x].children[0].setAttribute('class','')
      },
      /**
       * 遊戲結束時按鍵事件
       * @param {Event}} e 
       */
      again(e) {
        let _this = this
        let type = ''
        e.keyCode ? type = e.keyCode : type = e
        if (type == 89) {
          _this.status = 'play'
          _this.score = 0
          _this.speed = 400
          _this.alive = true
          _this.positionRecord = [{
            x: Math.round(this.width / 2),
            y: Math.round(this.height / 2)
          }]
          _this.$nextTick(() => {
            document.addEventListener('keydown', _this.keycodeControl)
            document.addEventListener('touchstart', _this.touch, {
              passive: false
            });
            document.addEventListener('touchmove', _this.touch, {
              passive: false
            });
            document.addEventListener('touchend', _this.touch, {
              passive: false
            });
            _this.food()
            _this.drawSnake()
          })
        } else if (type == 78) {
          _this.status = 'start'
          _this.score = 0
          _this.speed = 400
          _this.alive = true
          _this.positionRecord = [{
            x: Math.round(this.width / 2),
            y: Math.round(this.height / 2)
          }]
          _this.$nextTick(() => {
            document.removeEventListener('keydown', _this.again)
            document.addEventListener('keydown', _this.start)
          })
        }
      },
      stop() {
        console.log(this.interval)
        clearInterval(this.interval)
        console.log(this.interval)
      }
    }
  }
</script>

<style lang="scss">
@import "~bootstrap/scss/bootstrap";
</style>