<template>
  <div
    ref="container"
    class="scroll-container"
    @scroll="handleMove"
  >
    <div
      class="scroll-container__wrap"
      :style="containerPadding"
    >
      <div
        v-for="item in dataShowList"
        :key="item.id"
        class="scroll-container__item"
        :style="{ height: `${itemHeight}px` }"
      >
        <slot :row="item" />
      </div>
      <p v-if="loading" class="loading">loading...</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dataList: [],
      dataShowList: [],
      // loading: false,
      pageSize: 0,

      totalItemNum: 0,
      rowInScreenNum: 0,

      // itemHeight: 150,
      wrapHeight: 0,
      screenItemHeight: 0,

      lastScrollTop: 0,

      fromIndex: 0,
      toIndex: 0,

      lastPaddingObj: null
    }
  },

  props: {
    fnFetch: {
      type: Function,
      required: true
    },
    itemHeight: {
      type: Number,
      default: 50
    },
    loading: {
      type: Boolean,
      required: true,
      default: false
    }
  },

  computed: {
    containerPadding() {
      const paddingTop = this.fromIndex * this.itemHeight
      if (this.loading && this.lastPaddingObj) {
        this.lastPaddingObj.paddingBottom = 0
      } else {
        this.lastPaddingObj = {
          paddingTop: `${paddingTop}px`,
          // paddingBottom: `${this.wrapHeight - paddingTop - this.itemHeight * (this.toIndex - this.fromIndex)}px`
          paddingBottom: `${this.wrapHeight - paddingTop - this.itemHeight * this.pageSize}px`
        }
      }
      return this.lastPaddingObj
    }
  },

  created() {
    this.getData()
  },

  mounted() {
    const containerHeight = this.$refs.container.offsetHeight
    this.rowInScreenNum = Math.ceil(containerHeight / this.itemHeight)
    this.screenItemHeight = this.rowInScreenNum * this.itemHeight

    this.pageSize = this.toIndex = this.rowInScreenNum * 2

    if (this.pageSize < this.toIndex) {
      throw `当前滚动容器内一屏渲染${this.rowInScreenNum}条，pageSize必须至少为它的2倍14条`
    }
  },

  methods: {
    changeLoading(val) {
      this.$emit('update:loading', val)
    },

    getData() {
      // this.loading = true
      this.fetch().then(data => {
        this.changeLoading(true)
        // this.loading = false
        this.dataList = this.dataList.concat(data)
        this.totalItemNum = this.dataList.length
        this.setSliceData()

        this.wrapHeight = this.totalItemNum * this.itemHeight
      })
    },

    fetch() {
      return this.fnFetch(this.dataList).then(res => {
        // 校验res是否为数组
        if (!Array.isArray(res)) {
          throw 'fn-fetch方法resolve的data必须为数组'
        }
        return res
      })
    },

    setSliceData() {
      // console.log(this.fromIndex, this.toIndex)
      this.dataShowList = this.dataList.slice(this.fromIndex, this.toIndex)
    },

    handleMove(e) {
      const { scrollTop } = e.target

      if (this.loading) {
        return
      }

      if (
        Math.abs(scrollTop - this.lastScrollTop) > this.screenItemHeight
      ) {
        this.lastScrollTop = scrollTop
        const count = Math.floor(this.lastScrollTop / this.screenItemHeight)
        if (count > 0) {
          this.fromIndex = count * this.rowInScreenNum
          this.toIndex = this.fromIndex + this.pageSize
          if (this.toIndex >= this.dataList.length) {
            // 重新拉取新数据
            console.log('get data, this.fromIndex', this.fromIndex, 'this.toIndex', this.toIndex)
            this.getData()
          } else {
            this.setSliceData()
          }
        }
      } else {
        const count = Math.floor(this.lastScrollTop / scrollTop)
        if (count > 0) {
          console.log('up count', count)
          this.lastScrollTop -= this.screenItemHeight
          if (this.lastScrollTop < this.screenItemHeight) {
            this.lastScrollTop = 0
          }
          this.fromIndex -= count * this.rowInScreenNum
          if (this.fromIndex < 0) {
            this.fromIndex = 0
          }
          this.toIndex = this.fromIndex + this.pageSize
          this.setSliceData()
        }
      }
    }
  }
}
</script>

<style>
  .scroll-container {
    height: 100%;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
  }
  .scroll-container__item {
    display: flex;
    align-items: center;
    /*height: 150px;*/
    border-bottom: 1px #cecece solid;
    padding-left: 10px;
    overflow: hidden;
  }
  .loading {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 50px;
  }
</style>
