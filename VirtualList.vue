<template>
    <!-- 可视区域的容器 -->
    <div ref="list" class="infinite-list-container" @scroll="scrollEvent($event)">
    <div class="infinite-list-phantom" :style="{
        height: `${listHeight}px`
    }"></div>
    <div class="infinite-list" :style="{ transform: getTransform }">
      <div ref="items"
        class="infinite-list-item" 
        v-for="item in visibleData" 
        :key="item.id"
        :style="{
            height: `${itemSize}px`,
        }"
      >{{ item.value }}</div>
    </div>
  </div>
</template>
<script>
let arr = [];
for (let i = 0; i < 1000; i++) {
  arr.push({ id: i, value: i });
}
export default {
    name: 'VirtualList',

    props:{
         //所有列表数据
        listData: {
            type: Array,
            default: () => arr,
        },
        //每项高度
            itemSize: {
            type: Number,
            default: 200,
        },
    },
    computed: {
        getTransform() {
            return `translate3d(0, ${this.startOffset}px,0)`
        },
        listHeight() {
            return this.listData.length * this.itemSize
        },
        visibleCount() {
            return Math.ceil(this.screenHeight / this.itemSize)
        },
        visibleData() {
            return this.listData.slice(this.start,Math.min(this.end, this.listData.length))
        },
    },
    data() {
        return {
            screenHeight: 0,
            startOffset: 0,
            start: 0,
            end: null,
        }
    },
    mounted() {
        this.screenHeight = this.$el.clientHeight
        this.start = 0
        this.end = this.start + this.visibleCount
    },
    methods: {
        scrollEvent() {
            let scrollTop = this.$refs.list.scrollTop

            this.start = Math.floor(scrollTop / this.itemSize)

            this.end = this.start + this.visibleCount

            this.startOffset = scrollTop - (scrollTop % this.itemSize)
            console.log('startOffset',scrollTop,  (scrollTop % this.itemSize), this.startOffset)
        },

    },
}
</script>
<style>
.infinite-list-container {
  height: 100vh;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
  border: 5px solid red;
}

.infinite-list-phantom {
  position: absolute;
  background: pink;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.infinite-list {
  border: 10px solid green;
  left: 0;
  right: 0;
  /* top: 0; */
  position: absolute;
  text-align: center;
}

.infinite-list-item {
  padding: 10px;
  height: 200px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}
    
</style>