<template>
  <div id="app">
    <div v-show="foodList.length < 9">
      <nut-textinput
        v-model="currentVal"
        label="今天什么："
        placeholder="请输入想吃的饭"
        :clearBtn="true"
        :disabled="false"
      />
      <nut-button
        @click="add"
        style="width: 100%;margin-top:10px"
      >
        确定
      </nut-button>
    </div>
    <div class="food-list">
      <div class="food-list-item" v-for="(item, index) in foodList" :key="index">
        <span>{{item}}</span>
        <span class="delete" @click="deleteCurrent(index)">删除</span>
      </div>
    </div>
    <div class="luck" :class="{'show-luck': foodList.length > 1}">
      <nut-luckdraw
        class="drawTable"
        ref="luckDrawPrize"
        :luck-width="luckWidth"
        :luck-height="luckheight"
        :prize-list="foodList"
        :turns-number="turnsNumber"
        :turns-time="turnsTime"
        :prize-index="prizeIndex"
        :style-opt="styleOpt"
        @end-turns="endTurns"
      >
        <template slot="item" slot-scope="scope">
          <div class="drawTable-name">{{ scope.item }}</div>
          <!-- <div class="drawTable-img"> -->
            <!-- <img :src="scope.item.prizeImg"> -->
          <!-- </div> -->
        </template>
      </nut-luckdraw>
      <div @click="startTurns" class="pointer" :style="pointerStyle"></div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Home',
  components: {
  },
  mounted () {
    this.foodList = localStorage.foodList ? JSON.parse(localStorage.foodList) : []
  },
  data () {
    return {
      currentVal: '',
      foodList: [],
      // 转盘大小
      luckWidth: '300px',
      luckheight: '300px',
      // 转盘指针图片样式
      pointerStyle: {
        width: '80px',
        height: '80px',
        backgroundImage: 'url("https://img11.360buyimg.com/imagetools/jfs/t1/89512/11/15244/137408/5e6f15edEf57fa3ff/cb57747119b3bf89.png")',
        backgroundSize: 'contain',
        backgroundRepeat: 'no-repeat'
      },
      turnsNumber: 5, // 转动圈数
      turnsTime: 5, // 转动时间：S
      styleOpt: {
        prizeBgColors: ['rgb(255, 231, 149)', 'rgb(255, 247, 223)', 'rgb(255, 231, 149)', 'rgb(255, 247, 223)', 'rgb(255, 231, 149)', 'rgb(255, 247, 223)'],
        borderColor: '#ff9800'
      },
      prizeIndex: -1, // 中奖奖品的index
      lock: false, // 防止多次连续点击抽奖
      num: 5 // 抽奖次数,根据需求定义
    }
  },
  methods: {
    deleteCurrent (index) {
      this.foodList.splice(index, 1)
    },
    add () {
      if (!this.currentVal) {
        this.$toast.fail('请输入要吃的实物')
        return
      }
      const hasFood = this.foodList.find(item => item === this.currentVal)
      if (hasFood) {
        this.$toast.fail('已有该食物')
        return
      }
      this.foodList.push(this.currentVal)
      // 添加本地存储
      localStorage.foodList = JSON.stringify(this.foodList)
      this.currentVal = ''
    },
    startTurns () {
      this.lock = true
      // 此为模拟随机数字，这里可以接受后台中奖信息
      const index = Math.floor(Math.random() * this.foodList.length)
      // 成功后抽奖次数减1
      this.num--
      // 中奖奖品的index
      this.prizeIndex = index
      // 调用组件的方法，使转盘转动并停留在中奖奖品的那个扇形区域
      this.$refs.luckDrawPrize.rotate(index)
    },
    endTurns () {
      this.$dialog({
        content: `今天吃${this.foodList[this.prizeIndex]}`,
        noCloseBtn: false,
        noOkBtn: true,
        cancelBtnTxt: '我知道了'
      })
      this.lock = false
    }
  }
}
</script>
<style lang="scss" scoped>
.food-list {
  padding-top: 10px;
  color: #444;
  &-item {
    padding: 5px 0;
    display: flex;
    justify-content: space-between;
    .delete {
      padding: 4px;
      font-size: 13px;
      color: #ff4f18;
    }
  }
}
.luck {
  position: relative;
  margin-top: 170px;
  opacity: 0;
  &.show-luck {
    opacity: 1;
  }
}

</style>
