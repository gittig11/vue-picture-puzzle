<template>
  <div class="index">
    <div class="contain">
      <canvas class="can" ref="can"></canvas>
      <div v-if="isWin" class="win">恭喜成功！</div>
      <div class="btns">
        <span @click="newGame">重新游戏</span>
        <span @click="maskShow = true">编辑游戏</span>
        <span @click="isWinHandler">检验</span>
        <span>{{ step }}</span>
      </div>
      <img ref="img" class="img" src="../../public/test.png" alt="error" />

      <div v-show="maskShow" class="mask">
        行：<input type="text" v-model="row" placeholder="请输入行数" />
        列：<input type="text" v-model="column" placeholder="请输入列数" />
        <button @click="maskShow = false">完成</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      trueNum: 0, // 拼图位置正确的数量
      // row:拼图的总行数，column:拼图的总列数
      row: 3,
      column: 3,
      // 随机打乱的次数
      num: 100,
      // pic:拼图的所有子集和；
      // 元素：index：子图片的位置编号
      // row/column:对原图分割后的横纵编号
      // x/y:在canvas中的坐标位置（不会改变）
      pic: [{ x: 0, y: 0, row: 0, column: 0, index: 0 }],
      sign: 15,
      isWin: false,
      step: 0,
      maskShow: false
    }
  },
  mounted() {
    this.newGame()
    document.onkeydown = event => {
      let key = event.keyCode
      if (key === 38 || key === 87) this.moveHandler('up')
      else if (key === 40 || key === 83) this.moveHandler('down')
      else if (key === 37 || key === 65) this.moveHandler('left')
      else if (key === 39 || key === 68) this.moveHandler('right')
    }
  },
  methods: {
    // 判断是否完成拼图
    isWinHandler() {
      this.trueNum = 0
      console.log(this.pic)
      for (let i = 0; i < this.row * this.column; i++) {
        if (
          this.pic[i].x === this.pic[i].row &&
          this.pic[i].y === this.pic[i].column
        ) {
          this.trueNum += 1
          // this.isWin = true
          // this.step = 0
        }
      }
      console.log(this.trueNum)
      if (this.trueNum === 9) {
        this.isWin = true
        this.step = 0
      }
    },
    // 移动的函数方法
    moveHandler(dir) {
      let re
      let temp = {}
      if (dir === 'down' && this.pic[this.sign].x > 0) {
        // 根据空白块的row和column推算出上面一块图片的序号
        // 在将两个图片快进行互换位置，及交换row、column、index
        // 重新赋值this.sign（标志着空白块的序号：默认15）
        re = (this.pic[this.sign].x - 1) * this.row + this.pic[this.sign].y
        temp.row = this.pic[re].row
        temp.column = this.pic[re].column
        temp.index = this.pic[re].index
        this.pic[re] = {
          ...this.pic[re],
          row: this.pic[this.sign].row,
          column: this.pic[this.sign].column,
          index: this.pic[this.sign].index
        }
        this.pic[this.sign] = { ...this.pic[this.sign], ...temp }
        this.step = this.step + 1
      } else if (dir === 'up' && this.pic[this.sign].x < this.row - 1) {
        re = (this.pic[this.sign].x + 1) * this.row + this.pic[this.sign].y
        temp.row = this.pic[re].row
        temp.column = this.pic[re].column
        temp.index = this.pic[re].index
        this.pic[re] = {
          ...this.pic[re],
          row: this.pic[this.sign].row,
          column: this.pic[this.sign].column,
          index: this.pic[this.sign].index
        }
        this.pic[this.sign] = { ...this.pic[this.sign], ...temp }
        this.step = this.step + 1
      } else if (dir === 'right' && this.pic[this.sign].y > 0) {
        re = this.pic[this.sign].x * this.row + this.pic[this.sign].y - 1
        temp.row = this.pic[re].row
        temp.column = this.pic[re].column
        temp.index = this.pic[re].index
        this.pic[re] = {
          ...this.pic[re],
          row: this.pic[this.sign].row,
          column: this.pic[this.sign].column,
          index: this.pic[this.sign].index
        }
        this.pic[this.sign] = { ...this.pic[this.sign], ...temp }
        this.step = this.step + 1
      } else if (dir === 'left' && this.pic[this.sign].y < this.column - 1) {
        re = this.pic[this.sign].x * this.row + this.pic[this.sign].y + 1
        temp.row = this.pic[re].row
        temp.column = this.pic[re].column
        temp.index = this.pic[re].index
        this.pic[re] = {
          ...this.pic[re],
          row: this.pic[this.sign].row,
          column: this.pic[this.sign].column,
          index: this.pic[this.sign].index
        }
        this.pic[this.sign] = { ...this.pic[this.sign], ...temp }
        this.step = this.step + 1
      }
      this.drawHandler()
    },
    // 绘制拼图
    drawHandler() {
      let canvas = this.$refs.can
      let ctx = canvas.getContext('2d')
      canvas.width = 400
      canvas.height = 400
      ctx.clearRect(0, 0, 400, 400)
      let width = canvas.width / this.column
      let height = canvas.width / this.row
      let img = new Image()
      img.src = require('../../public/test.png')
      img.onload = () => {
        for (let i = 0; i < this.row * this.column; i++) {
          // 绘制到canvas的各元素的起始坐标
          let dx = this.pic[i].y * width
          let dy = this.pic[i].x * height
          // 对图片进行切割的起始点坐标
          let cx = this.pic[i].column * width
          let cy = this.pic[i].row * height
          ctx.drawImage(img, cx, cy, width, height, dx, dy, width, height)
          if (this.pic[i].index === -1) {
            this.sign = i
            ctx.clearRect(dx, dy, width, height)
          }
        }
      }
      // this.isWinHandler()
    },
    // 获取随机排序
    randomHandler() {
      this.trueNum = 0
      for (let i = 0; i < this.row * this.column; i++) {
        // 设置切割后每个小图片的位置
        let row = parseInt(i / this.row)
        let column = i - row * this.column
        // 对在canvas的排列进行初始化，不会进行改变
        let x = parseInt(i / this.row)
        let y = i - x * this.column
        this.pic[i] = {
          ...this.pic[i],
          x: x,
          y: y,
          row: row,
          column: column,
          index: i
        }
        if (i === this.row * this.column - 1) {
          this.pic[i] = { ...this.pic[i], row: row, column: column, index: -1 }
        }
      }
      for (let i = 0; i < this.num; i++) {
        let ran1
        let ran2
        let temp = {}
        // 随机获取0-14
        ran1 = parseInt((this.row * this.column - 1) * Math.random())
        ran2 = parseInt((this.row * this.column - 1) * Math.random())
        temp.row = this.pic[ran1].row
        temp.column = this.pic[ran1].column
        this.pic[ran1] = {
          ...this.pic[ran1],
          row: this.pic[ran2].row,
          column: this.pic[ran2].column
        }
        this.pic[ran2] = { ...this.pic[ran2], ...temp }
      }
    },
    newGame() {
      this.isWin = false
      this.step = 0
      this.randomHandler()
      this.drawHandler()
    }
  }
}
</script>

<style scoped lang="scss">
.mask {
  width: 200px;
  height: 200px;
  background-color: rosybrown;
  position: absolute;
  left: 510px;
  top: 0;
}
.btns > span {
  display: inline-block;
  width: 80px;
  height: 24px;
  font-size: 12px;
  text-align: center;
  line-height: 24px;
  margin-bottom: 5px;
  background-color: thistle;
  cursor: pointer;
  &:last-child{
    margin-bottom: 0;
  }
}
.btns {
  width: 80px;
  border: 1px solid tan;
  border-radius: 5px;
  background-origin: border-box;
  padding: 5px;
  position: absolute;
  right: 60px;
  top: 20px;
}
.win {
  width: 402px;
  height: 402px;
  line-height: 402px;
  text-align: center;
  font-size: 24px;
  opacity: 0.5;
  background-color: paleturquoise;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
}
.img {
  // display: inline-block;
  position: absolute;
  right: 10px;
  top: 170px;
  width: 200px;
  height: 200px;
}
.can {
  border: 1px solid teal;
}
.contain {
  position: relative;
}
</style>
