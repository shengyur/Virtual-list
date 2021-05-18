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
        estimatedItemSize: {
            type: Number,
        },
    },
    computed: {
        getTransform() {
            return `translate3d(0, ${this.startOffset}px,0)`
        },
        listHeight() {
            // return this.listData.length * this.itemSize
            return this.positions[this.positions.length-1].bottom
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
            positions: [

            ]
        }
    },
    updated() {
        let nodes = this.$refs.items
        nodes.forEach((node) => {
            let rect = node.getBoundingClientRect()
            let height = rect.height
            let index = + node.id.slice(1)
            let oldHeight = this.positions[index].height
            let updateValue = oldHeight - height
            if (updateValue) {
                this.positions[index].bottom = this.positions[index].bottom - updateValue
                this.positions[index].height = height
                for(let k= index +1; k< this.positions.length; k++){
                    this.positions[k].top = this.positions[k-1].bottom
                    this.positions[k].bottom = this.positions[k].bottom - updateValue
                }
            }
        })
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

            // this.startOffset = scrollTop - (scrollTop % this.itemSize)

            console.log('startOffset',scrollTop,  (scrollTop % this.itemSize), this.startOffset)
            if(this.start >= 1){
                this.startOffset = this.positions[this.start - 1].bottom
            }else{
                this.startOffset = 0;
            }
        },
        initPositions() {
            this.positions = this.listData.map((item,index) => {
                return {
                    index,
                    height: this.estimatedItemSize,
                    top: index * this.estimatedItemSize,
                    bottom: (index + 1) * this.estimatedItemSize
                }
            })
        },
        //二分法查找
        binarySearch(list,value){
            let start = 0;
            let end = list.length - 1;
            let tempIndex = null;
            while(start <= end){
                let midIndex = parseInt((start + end)/2);
                let midValue = list[midIndex].bottom;
                if(midValue === value){
                return midIndex + 1;
                }else if(midValue < value){
                start = midIndex + 1;
                }else if(midValue > value){
                if(tempIndex === null || tempIndex > midIndex){
                    tempIndex = midIndex;
                }
                end = end - 1;
                }
            }
            return tempIndex;
        },
        //获取列表起始索引
        getStartIndex(scrollTop = 0){
            //二分法查找
            return this.binarySearch(this.positions,scrollTop)
            // let item = this.positions.find(i => i && i.bottom > scrollTop);
            // return item.index;
        }
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