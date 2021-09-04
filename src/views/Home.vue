<template>
  <div class="content">
    <ul class="puzzle-box" @click="goDetail">
      <li v-for="(item, index) in puzzles" :key="index"
        class="puzzle"
        :class="{'no-bgc': item === ''}"
        :data-index="index"
      >
        {{item}}
      </li>
    </ul>

    <p class="inps">
      <input type="radio" id="one" :value="0" v-model="picked">
      <label for="one">有序打乱</label>
      <input type="radio" id="two" :value="1" v-model="picked">
      <label for="two">随机打乱</label>
    </p>

    <p class="btn" @click="init">重新开始</p>
    <p> 耗时：{{time}} s </p>

    <div class="selected-box">
      <span>选择类型：</span>
      <select v-model="boxLen" class="myselect">
        <option v-for="(item, i) in options" :key="i" :value="item.value">
          {{ item.text }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

// 参考：
// 利用Vue.js实现拼图游戏 - 劳卜 - 博客园
// https://www.cnblogs.com/luozhihao/p/5726661.html

export default {
  name: 'Home',
  data() {
    return {
      options: [
        { text: '3*3', value: 3 },
        { text: '4*4', value: 4 },
        { text: '5*5', value: 5 },
        { text: '6*6', value: 6 }
      ],

      boxLen: 4, // 正方形拼图的边长
      picked: 0,
      puzzles: [],
      initNum: 15, // ''的初始位置
      time: 0,
      timer: null,
      randomTimes: 200, // 随机次数
      isFirstClick: true, // 第一次点击
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    getNextNum(num) { // num: ''位置，如15
      let increaseArr = [num + this.boxLen, num + 1, num - 1, num - this.boxLen]
      let arr = increaseArr.filter(val => this.isIndexValid(val))
      let index = Math.floor(Math.random() * arr.length)
      this.initNum = arr[index]
      // console.log(increaseArr)
      // console.log(arr)
      // console.log(arr[index])
      return arr[index]
    },

    isIndexValid(val) {
      return val >= 0 && val < this.boxLen ** 2
    },

    init() {
      // 设置盒子宽度
      let puzzleBox = document.querySelector('.puzzle-box')
      puzzleBox.style.width = `${this.boxLen * 100}px`
      // 更新 this.initNum
      this.initNum = this.boxLen ** 2 - 1
      this.randomTimes = 200 * (this.boxLen - 3) || 100 // 随机次数随着拼图的增大而增加，最小次数是100次

      clearInterval(this.timer)
      this.time = 0

      // 随机打乱
      this.randomArr()

      // 点击"重新开始"按钮，重新计时
      this.isFirstClick = true
    },

    // 计时
    countFn() {
      this.timer = setInterval(() => {
        this.time += 1
      }, 1000)
    },

    randomArr() {
      // 初始化数组
      let arr = []
      for (let index = 1; index < this.boxLen ** 2; index++) {
        arr.push(index)
      }
      arr.push('')
      this.puzzles = arr

      if (this.picked === 0) {
        // 按照一定规则乱序
        for (let index = 0; index < this.randomTimes; index++) {
          let num = this.getNextNum(this.initNum)
          this.goDetail('', num)
        }
      } else {
        console.log('随机乱序')
        // 随机乱序
        this.puzzles = this.puzzles.sort(() => Math.random() - 0.5)
      }
    },

    swap(arr, i, j, num1, num2) {
      this.$set(arr, i, num1)
      this.$set(arr, j, num2)
    },

    goDetail(e, num) {
      if (num >= 0) {
        // console.log(num)
        this.changeByIndex(num)
        return
      }
      // 第一次点击后才开始计时
      if (this.isFirstClick) {
        this.isFirstClick = false
        this.countFn()
      }
      if (e.target.nodeName.toLowerCase() === 'li') {
        const index = parseInt(e.target.dataset.index)
        // console.log(index)

        this.changeByIndex(index)
        this.check()
      }
    },

    changeByIndex(index) {
      let curNum = this.puzzles[index]
      let leftNum = this.puzzles[index - 1]
      let rightNum = this.puzzles[index + 1]
      let topNum = this.puzzles[index - this.boxLen]
      let bottomNum = this.puzzles[index + this.boxLen]
      if (curNum === '') {
        return
      }
      let num = this.puzzles.indexOf('')
      // console.log(curNum, leftNum, rightNum, topNum, bottomNum)
      // console.log(num)
      if (!(num === index - 1 || num === index + 1 || num === index - this.boxLen || num === index + this.boxLen)) {
        // console.log("can't move.")
        return
      }

      if (leftNum === '' && index % this.boxLen) { // index为8时，格子8和9不能交换
        this.swap(this.puzzles, index - 1, index, curNum, '')
      } else if (rightNum === '' && index % this.boxLen !== (this.boxLen - 1)) { // index为7时，格子8和9不能交换
        this.swap(this.puzzles, index + 1, index, curNum, '')
      } else if (topNum === '') {
        this.swap(this.puzzles, index - this.boxLen, index, curNum, '')
      } else if (bottomNum === '') {
        this.swap(this.puzzles, index + this.boxLen, index, curNum, '')
      }
      // console.log(this.puzzles)
    },

    check() {
      if (this.puzzles[this.boxLen ** 2 - 1] === '') {
        const newPuzzles = this.puzzles.slice(0, this.boxLen ** 2 - 1)
        const isPass = newPuzzles.every((val, i) => val === i + 1)
        if (isPass) {
          setTimeout(() => {
            alert(`祝贺成功！耗时${this.time} s.`)
            clearInterval(this.timer)
          }, 100)
        }
      }
    },
  },
  watch: {
    boxLen() {
      this.init()
    },
  },
}
</script>

<style scoped lang="scss">
  *{
    margin: 0;
    padding: 0;
  }
  ul, li{
    list-style: none;
  }
  .content{
    .puzzle-box{
      display: flex;
      align-items: center;
      flex-wrap: wrap;
      width: 400px;
      border-radius: 5px;
      margin: 0 auto;
      border: 2px solid #ccc;
      .puzzle{
        user-select: none;
        width: 100px;
        height: 100px;
        border-radius: 5px;
        flex-shrink: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: #96c6f4;
        border: 1px solid #ccc;
        box-sizing: border-box;
        font-size: 24px;
        font-weight: 500;
      }
      .no-bgc{
        background-color: #fff;
        border: none;
      }
    }

    .inps{
      margin: 10px 0;
      input{
        margin: 0 5px 0 15px;
      }
    }
    .btn{
      display: flex;
      align-items: center;
      justify-content: center;
      margin: 0 auto 10px;
      background-color: #c5dbef;
      padding: 5px;
      width: 100px;
      height: 30px;
      border-radius: 5px;
      &:active{
        background-color: #96c6f4;
      }
    }

    .selected-box{
      margin: 10px 0;
    }
    .myselect{
      font-size: 16px;
    }
  }
</style>
