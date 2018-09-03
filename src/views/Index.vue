<template>
  <div class="demo">
    <div style="text-align: center;">
      <p style="font-size: 30px;padding-top: 50px;">Vue列表无限滚动的解决方案</p>
      <p style="color: #999;margin: -30px 0 50px 0">
        本文思路原理借鉴 <a href="https://juejin.im/entry/5819993fbf22ec0068aab054">Vue.js 一个超长列表无限滚动加载的解决方案</a><br/>
        本例子为更简单实现，用2个区域实现，其是三个区域个人觉得不便于理解
      </p>
    </div>
    <div class="vue-list">
      <div class="vue-list__wrap">
        <infinite-scrolling
          :fn-fetch="fetch"
          :item-height="70"
          :loading="loading"
        >
        <span slot-scope="{ row }">
          {{ row.name }}
        </span>
        </infinite-scrolling>
      </div>

      <div class="other-info" v-if="$children[0]">
        <p style="font-size: 30px">data present</p>
        <p>is getting data: {{ loading }}</p>
        <p>dataList: Array[{{ $children[0].dataList.length }}]</p>
        <p>dataShowList: Array[{{ $children[0].dataShowList.length }}]</p>
        <p>fromIndex: {{ $children[0].fromIndex }}</p>
        <p>toIndex: {{ $children[0].toIndex }}</p>
        <p>pageSize: {{ $children[0].pageSize }}</p>
        <p>wrapHeight: {{ $children[0].wrapHeight }}px</p>
        <p>screenItemHeight: {{ $children[0].screenItemHeight }}px</p>
        <p>totalItemNum: {{ $children[0].totalItemNum }}</p>
        <p>rowInScreenNum: {{ $children[0].rowInScreenNum }}</p>
      </div>
    </div>
    <p style="padding-top: 30px">
      git地址：<a href="">infinite-scrolling</a> |
      文章详解：<a href="">列表无限滚动的优化方案</a>
    </p>
  </div>
</template>

<script>
import InfiniteScrolling from '../components/InfiniteScrolling'

export default {
  components: {
    InfiniteScrolling
  },

  data() {
    return {
      loading: false
    }
  },

  methods: {
    fetch(dataList) {
      this.loading = true
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          const res = Array.from(new Array(20)).fill(1)
          const data = []
          res.forEach((item, i) => {
            data.push({ name: `item ${i + 1 + dataList.length}`, id: i + 1 + dataList.length })
          })
          this.loading = false
          resolve(data)
        }, 1000)
      })
    }
  }
}
</script>

<style>
  .demo {
    display: flex;
    /*justify-content: center;*/
    flex-direction: column;
    align-items: center;
    height: 100%;
    background-color: #f2f2f2;
  }
  .vue-list {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .vue-list__wrap {
    width: 320px;
    height: 480px;
    background-color: #fff;
  }
  .other-info {
    margin-left: 100px;
  }
</style>
