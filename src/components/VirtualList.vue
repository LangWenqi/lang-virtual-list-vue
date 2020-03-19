<template>
  <div ref="list" :style="{height}" :class="hideScrollBar ? 'infinite-list-container-no-scroll' : 'infinite-list-container'" 
    @scroll="scrollEvent($event)"
  >
    <div ref="phantom" class="infinite-list-phantom"></div>
    <div ref="top" v-if="topLoadMore" v-show="dragState !== 'none' && touchDistance >= 20" class="infinite-top-container">
      <slot name="top" :dragState="dragState" :dargDistance="touchDistance">
        <div class="infinite-top-content" :style="{color:textColor}">
          <span class="infinite-top-content-icon icon-arrow" :class="dragState === 'drop' ? 'icon-reverse':''" v-if="dragState === 'pull' || dragState === 'drop'">
            <svg t="1572934878285" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4245" width="24" height="24">
              <path d="M548.352 241.152L716.8 409.6a32.768 32.768 0 0 1 0 46.592 30.72 30.72 0 0 1-45.568 0l-116.736-115.2v464.896a32.768 32.768 0 1 1-65.024 0V340.992L372.736 460.8a39.424 39.424 0 0 1-45.568-6.656 32.768 32.768 0 0 1 0-46.592l162.816-166.4a35.328 35.328 0 0 1 58.368 0z" :fill="textColor" p-id="4246"></path>
            </svg>
          </span>
          <span class="infinite-top-content-icon icon-revolve" v-if="dragState === 'loading'">
            <svg class="icon" t="1572936012117" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="9195" width="32" height="32">
              <path d="M527.36 225.28h-30.72c-8.192 0-15.36 7.168-15.36 15.36v133.12c0 8.192 7.168 15.36 15.36 15.36h30.72c8.192 0 15.36-7.168 15.36-15.36v-133.12c0-8.192-7.168-15.36-15.36-15.36z m256 256h-133.12c-8.192 0-15.36 7.168-15.36 15.36v30.72c0 8.192 7.168 15.36 15.36 15.36h133.12c8.192 0 15.36-7.168 15.36-15.36v-30.72c0-8.192-7.168-15.36-15.36-15.36z m-256 153.6h-30.72c-8.192 0-15.36 7.168-15.36 15.36v133.12c0 8.192 7.168 15.36 15.36 15.36h30.72c8.192 0 15.36-7.168 15.36-15.36v-133.12c0-8.192-7.168-15.36-15.36-15.36zM389.12 527.36v-30.72c0-8.192-7.168-15.36-15.36-15.36h-133.12c-8.192 0-15.36 7.168-15.36 15.36v30.72c0 8.192 7.168 15.36 15.36 15.36h133.12c8.192 0 15.36-7.168 15.36-15.36z m220.16-91.136c6.144 6.144 15.36 6.144 21.504 0l94.208-94.208c6.144-6.144 6.144-15.36 0-21.504l-21.504-21.504c-6.144-6.144-15.36-6.144-21.504 0l-94.208 94.208c-6.144 6.144-6.144 15.36 0 21.504l21.504 21.504z m22.528 151.552c-6.144-6.144-15.36-6.144-21.504 0l-21.504 21.504c-6.144 6.144-6.144 15.36 0 21.504l94.208 94.208c6.144 6.144 15.36 6.144 21.504 0l21.504-21.504c6.144-6.144 6.144-15.36 0-21.504l-94.208-94.208z m-217.088 0c-6.144-6.144-15.36-6.144-21.504 0l-94.208 94.208c-6.144 6.144-6.144 15.36 0 21.504l21.504 21.504c6.144 6.144 15.36 6.144 21.504 0l94.208-94.208c6.144-6.144 6.144-15.36 0-21.504l-21.504-21.504z m-72.704-289.792c-6.144-6.144-15.36-6.144-21.504 0l-21.504 21.504c-6.144 6.144-6.144 15.36 0 21.504l94.208 94.208c6.144 6.144 15.36 6.144 21.504 0l21.504-21.504c6.144-6.144 6.144-15.36 0-21.504l-94.208-94.208z" :fill="textColor" p-id="9196"></path>
            </svg>
          </span>
          <span class="infinite-top-content-title">
            <template v-if="dragState === 'pull'">
              {{topPullText}}
            </template>
            <template v-if="dragState === 'drop'">
               {{topDropText}}
            </template>
            <template v-if="dragState === 'loading'">
              {{topLoadingText}}
            </template>
          </span>
        </div>
      </slot>
    </div>
    <div ref="content" class="infinite-list">
      <div ref="items" class="infinite-list-item-container" :id="row._key" :key="row._key" v-for="(row) in visibleData">
        <template v-for="(item,col_index) in row.value">
          <div class="infinite-item" :key="row._key + '-' + col_index">
            <slot name="default" :item="item"  :index="row.index + col_index"></slot>
          </div> 
        </template>
        <template v-if="row.value.length < column">
          <div v-for="(item,index) in (column - row.value.length%column)" class="infinite-item" :key="'empty-' + index">
          </div> 
        </template>
      </div>
			<div class="infinite-bottom-content infinite-top-content" :style="{color: textColor}" v-if="bottomLoading">
				<span class="infinite-top-content-icon icon-revolve">
					<svg class="icon" t="1572936012117" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="9195" width="32" height="32">
						<path d="M527.36 225.28h-30.72c-8.192 0-15.36 7.168-15.36 15.36v133.12c0 8.192 7.168 15.36 15.36 15.36h30.72c8.192 0 15.36-7.168 15.36-15.36v-133.12c0-8.192-7.168-15.36-15.36-15.36z m256 256h-133.12c-8.192 0-15.36 7.168-15.36 15.36v30.72c0 8.192 7.168 15.36 15.36 15.36h133.12c8.192 0 15.36-7.168 15.36-15.36v-30.72c0-8.192-7.168-15.36-15.36-15.36z m-256 153.6h-30.72c-8.192 0-15.36 7.168-15.36 15.36v133.12c0 8.192 7.168 15.36 15.36 15.36h30.72c8.192 0 15.36-7.168 15.36-15.36v-133.12c0-8.192-7.168-15.36-15.36-15.36zM389.12 527.36v-30.72c0-8.192-7.168-15.36-15.36-15.36h-133.12c-8.192 0-15.36 7.168-15.36 15.36v30.72c0 8.192 7.168 15.36 15.36 15.36h133.12c8.192 0 15.36-7.168 15.36-15.36z m220.16-91.136c6.144 6.144 15.36 6.144 21.504 0l94.208-94.208c6.144-6.144 6.144-15.36 0-21.504l-21.504-21.504c-6.144-6.144-15.36-6.144-21.504 0l-94.208 94.208c-6.144 6.144-6.144 15.36 0 21.504l21.504 21.504z m22.528 151.552c-6.144-6.144-15.36-6.144-21.504 0l-21.504 21.504c-6.144 6.144-6.144 15.36 0 21.504l94.208 94.208c6.144 6.144 15.36 6.144 21.504 0l21.504-21.504c6.144-6.144 6.144-15.36 0-21.504l-94.208-94.208z m-217.088 0c-6.144-6.144-15.36-6.144-21.504 0l-94.208 94.208c-6.144 6.144-6.144 15.36 0 21.504l21.504 21.504c6.144 6.144 15.36 6.144 21.504 0l94.208-94.208c6.144-6.144 6.144-15.36 0-21.504l-21.504-21.504z m-72.704-289.792c-6.144-6.144-15.36-6.144-21.504 0l-21.504 21.504c-6.144 6.144-6.144 15.36 0 21.504l94.208 94.208c6.144 6.144 15.36 6.144 21.504 0l21.504-21.504c6.144-6.144 6.144-15.36 0-21.504l-94.208-94.208z" :fill="textColor" p-id="9196"></path>
					</svg>
				</span>
				<span class="infinite-top-content-title">
           {{bottomLoadingText}}
          </span>
			</div>
    </div>
  </div>
</template>


<script>
// 删除滚轮事件
import _ from '../util'
export default {
  name:'VirtualList',
  props: {
		// 是否隐藏滚动条
		hideScrollBar: {
			type: Boolean,
			default: false
		},
    //所有列表数据
    listData:{
      type:Array,
      default:()=>[]
    },
    //列表列数
    column:{
      type:Number,
      default:1
    },
    //是否开启下拉刷新
    topLoadMore:{
      type:Boolean,
      default:false
    },
    //是否开启下拉刷新
    scrollTopLoadMore:{
      type:Boolean,
      default:false
    },
		//是否开启下拉刷新
    bottomLoadMore:{
      type:Boolean,
      default:false
    },
    //超过阈值的回调
    topMethod:{
      type:Function,
      default () {
        return function(){}
      }
    },
		bottomMethod:{
      type:Function,
      default () {
        return function(){}
      }
    },
    scrollTopMethod:{
      type:Function,
      default () {
        return function(){}
      }
    },
    textColor:{
      type:String,
      default:'#000000'
    },
    topPullText:{
      type:String,
      default:'上拉刷新'
    },
    topDropText:{
      type:String,
      default:'释放更新'
    },
    topLoadingText:{
      type:String,
      default:'加载中...'
    },
		bottomLoadingText:{
      type:String,
      default:'加载中...'
    },
    //最大滑动距离
    maxDistance:{
      type:Number,
      default:0
    },
    //滑动距离与真实距离比值
    distanceScale :{
      type:Number,
      default:2
    },
    //滑动距离阈值，超过阈值回调
    topDistance :{
      type:Number,
      default:70
    },
    onScroll:{
      type:Function
    },
    onScrollEnd:{
      type:Function
    },
    //预估高度
    estimatedItemSize:{
      type:Number,
      required: true
    },
    //缓冲区比例
    bufferScale:{
      type:Number, 
      default:1
    },
    height:{
      type:String,
      default:'100%'
    }
  },
  data() {
    return {
      //拖拽状态
      // pull 开始拖拽，距离未达到topDistance
      // drop 距离达到 topDistance 释放触发 topMethod
      // loading 已被释放，topMethod 已经执行
      // none 拖拽完成或未触发
      dragState:'none', 
      bottomLoading: false,
      scrollTopLoading: false,
      //当前下拉距离
      touchDistance:0,
      //是否正在滚动
      scrolling:false,
      //可视区域高度
      screenHeight:0,
      //起始索引
      start:0,
      //结束索引
      end:0,
      moreNum: 0
    };
  },
  computed:{
    _listData(){
      return this.listData.reduce((init,cur,index)=>{
        if(index % this.column === 0){
          init.push({
            // _转换后的索引_第一项在原列表中的索引_本行包含几列
            _key:`_${index/this.column}_${index}_${this.column}`,
            value:[cur],
						index
          })
        }else{
          init[init.length - 1].value.push(cur)
        }
        return init;
      },[])
    },
    anchorPoint(){
      return this.positions.length ? this.positions[this.start] : null;
    },
    visibleCount(){
      return Math.ceil(this.screenHeight / this.estimatedItemSize);
    },
    aboveCount(){
      return Math.min(this.start,this.bufferScale * this.visibleCount);
    },
    belowCount(){
      return Math.min(this.listData.length - this.end,this.bufferScale * this.visibleCount);
    },
    visibleData(){
      let start = this.start - this.aboveCount;
      let end = this.end + this.belowCount;
      return this._listData.slice(start, end);
    }
  },
  watch:{
    // scrolling(cur,pre){
    //   //滚动结束
    //   if(!cur && pre){
        
    //   }
    // }
  },
  created(){
    this.initPositions();
    this.setScrollState(false);
  },
  mounted() {
    this.screenHeight = this.$el.clientHeight;
    this.start = 0;
    this.end = this.start + this.visibleCount;
    this.setStartOffset();

    //添加拖拽事件
    if(this.topLoadMore){
      this.$refs.list.addEventListener('touchstart',this.touchStartEvent)
      this.$refs.list.addEventListener('touchmove',this.touchMoveEvent)
      this.$refs.list.addEventListener('touchend',this.touchEndEvent)
    }
  },
  beforeDestroy(){
    if(this.topLoadMore){
      this.$refs.list.removeEventListener('touchstart',this.touchStartEvent)
      this.$refs.list.removeEventListener('touchmove',this.touchMoveEvent)
      this.$refs.list.removeEventListener('touchend',this.touchEndEvent)
    }
  },
  updated(){
    if(this.dragState !== 'none'){
      return;
    }
    //列表数据长度不等于缓存长度
    if(this._listData.length !== this.positions.length){
      this.initPositions();
    }
    this.$nextTick(function () {
      if(!this.$refs.items || !this.$refs.items.length){
        return ;
      }
      //获取真实元素大小，修改对应的尺寸缓存
      this.updateItemsSize(); 
      //更新列表总高度
      let height = this.positions[this.positions.length - 1].bottom;
      this.$refs.phantom.style.height = height + 'px'
      if (this.moreNum) {
        this.$refs.list.scrollTop = this.$refs.list.scrollTop + this.moreNum*this.estimatedItemSize
        this.moreNum = 0;
      }
      //更新真实偏移量
      this.setStartOffset();
    })
  },
  methods: {
    refreshComponent () {
      Object.assign(this.$data, this.$options.data());
      this.initPositions();
      this.setScrollState(false);
      if (this.$refs.list) {
        this.$refs.list.scrollTop = 0;
      }
      this.screenHeight = this.$el.clientHeight;
      this.start = 0;
      this.end = this.start + this.visibleCount;
      this.setStartOffset();
    },
    isToBottom () {
      return this.$refs.list.scrollHeight - this.$refs.list.scrollTop <= this.$refs.list.clientHeight;
    },
    isToTrueBottom () {
      return this.$refs.list.scrollTop > 0 && this.isToBottom();
    },
    isToTop () {
      return this.$refs.list.scrollTop <= 0;
    },
    getScrollTop () {
      return this.$refs.list.scrollTop;
    },
    getStart () {
      return this.start;
    },
    getEnd() {
      return this.end;
    },
    scrollToPosition (index) {
      this.$refs.list.scrollTop = this.positions[index] ? this.positions[index].top : 0;
    },
    scrollToTop () {
      this.$refs.list.scrollTop = 0;
    },
    scrollToBottom () {
      this.$refs.list.scrollTop = this.$refs.list.scrollHeight * (this.bufferScale + 1);
    },
    scrollTopLoadMoreFuc () {
			let scrollTop=this.$refs.list.scrollTop;
			let scrollHeight=this.$refs.list.scrollHeight;
			let offsetHeight=this.$refs.list.offsetHeight;
			this.scrollTopLoadMoreCallBack(scrollTop, scrollHeight, offsetHeight);
		},
		onscrollTopLoaded(moreNum) {
      this.scrollTopLoading = false;
      this.moreNum = moreNum;
		},
		scrollTopLoadMoreCallBack(scrollTop, scrollHeight, offsetHeight){
			if (scrollTop <= 0 && scrollHeight > offsetHeight && (!this.scrollTopLoading)) {
				this.scrollTopLoading = true;
				this.scrollTopMethod();
			}
		},
		bottomLoadMoreFuc () {
			let scrollTop=this.$refs.list.scrollTop;
			let scrollHeight=this.$refs.list.scrollHeight;
			let offsetHeight=this.$refs.list.offsetHeight;
			this.bottomLoadMoreCallBack(scrollTop, scrollHeight, offsetHeight);
		},
		onBottomLoaded() {
			this.bottomLoading = false;
		},
		bottomLoadMoreCallBack(scrollTop, scrollHeight, offsetHeight){
			if ((this._listData.length <= this.end + 1) && (!this.bottomLoading) && (Math.ceil(scrollTop) >= scrollHeight - offsetHeight)) {
				this.bottomLoading = true;
				this.bottomMethod();
			}
		},
    //设定滚动状态
    setScrollState(flg = false){
      this.scrolling = flg;
    },
    //防抖处理，设置滚动状态 
    scrollEnd:_.debounce(function(event,data){
      this.setScrollState(false);
      this.onScrollEnd && this.onScrollEnd(event, data);
			if (this.bottomLoadMore) {
				this.bottomLoadMoreFuc(); 
      }
      if (this.scrollTopLoadMore) {
        this.scrollTopLoadMoreFuc();
      }
    },100),
    scrollingEvent:function(event,data){
      this.onScroll && this.onScroll(event, data)
    },
    //初始化缓存
    initPositions(){
      this.positions = this._listData.map((d,index)=>({
          index,
          height:this.estimatedItemSize,
          top:index * this.estimatedItemSize,
          bottom:(index+1) * this.estimatedItemSize
        })
      );
    },
    //获取列表起始索引
    getStartIndex(scrollTop = 0){
      //二分法查找
      return this.binarySearch(this.positions,scrollTop)
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
    //获取列表项的当前尺寸
    updateItemsSize(){
      let nodes = this.$refs.items;
      nodes.forEach((node)=>{
        let rect = node.getBoundingClientRect();
        let height = rect.height;
        let index = +node.id.replace(/^_(\d+).*/,'$1')
        let oldHeight = this.positions[index].height;
        let dValue = oldHeight - height;
        //存在差值
        if(dValue){
          this.positions[index].bottom = this.positions[index].bottom - dValue;
          this.positions[index].height = height;
          
          for(let k = index + 1;k<this.positions.length; k++){
            this.positions[k].top = this.positions[k-1].bottom;
            this.positions[k].bottom = this.positions[k].bottom - dValue;
          }
        }
      })
    },
    //更新偏移量
    setStartOffset(){
      let startOffset;
      if(this.start >= 1){
        let size = this.positions[this.start].top - (this.positions[this.start - this.aboveCount] ? this.positions[this.start - this.aboveCount].top : 0);
        startOffset = this.positions[this.start - 1].bottom - size;
      }else{
        startOffset = 0;
      }
      this.startOffset = startOffset;
      this.$refs.content.style.transform = `translate3d(0, ${startOffset}px, 0)`
    },
    //滚动事件
    scrollEvent(event) {
      //当前滚动位置
      let scrollTop = this.$refs.list.scrollTop;
      //更新滚动状态
      this.setScrollState(true);
      //排除不需要计算的情况
      if(scrollTop > this.anchorPoint.bottom || scrollTop < this.anchorPoint.top){
        //此时的开始索引
        this.start = this.getStartIndex(scrollTop);
        //此时的结束索引
        this.end = this.start + this.visibleCount;
        //更新偏移量
        this.setStartOffset();
      }
      //触发外部滚动事件
      this.scrollingEvent(event,{
          start:this.start * this.column,
          end:Math.min(this.end * this.column,this.listData.length - 1),
          startOffset:this.startOffset,
          scrollTop
        }
      )
      //防抖处理滚动结束
      this.scrollEnd(event,{
          start:this.start * this.column,
          end:Math.min(this.end * this.column,this.listData.length - 1),
          startOffset:this.startOffset,
          scrollTop
        }
      );
    },
    //Start
    touchStartEvent(event){
      if(!this.topLoadMore){
        return;
      }
      //记录当前起始Y坐标
      this._startPos = event.touches[0].pageY;
      this._prevPos = event.touches[0].pageY;
    },
    //Move
    touchMoveEvent(event){
      if(!this.topLoadMore){
        return;
      }
      //暂时这样处理 loading 中不可滚动
      if(this.dragState === 'loading'){
        event.preventDefault();
        return ;
      }
      //当前Y坐标
      let curPos = event.touches[0].pageY;
      //下拉 且 不在顶部
      if(curPos > this._prevPos && this.$refs.list.scrollTop !== 0){
        return ;
      }
      //下拉 且 在顶部
      if(curPos > this._prevPos && this.$refs.list.scrollTop === 0){
        // event.preventDefault(); 
        this.touchDistance = Math.max(this.touchDistance + curPos - this._prevPos,0);
        
        let distance = ~~(this.touchDistance / this.distanceScale)

        //未达到阈值
        if(distance < this.topDistance){
          this.dragState = 'pull'
          this.$emit('top-status-change', this.dragState,distance)
        }
        //已达到阈值
        if(distance >= this.topDistance){
          this.dragState = 'drop'
          this.$emit('top-status-change', this.dragState,distance)
        }
        //设定偏移距离
        if(distance <= this.maxDistance || !this.maxDistance){
          let d = this.maxDistance ? Math.min(distance,this.maxDistance) : distance;
          setTimeout(()=>{
              this.$refs.content.style.transform = `translate3d(0,${d}px,0)`
              this.$refs.top.style.height = `${d}px`
          },0);
        }
      }
      //上划 且 没有拖拽距离
      if(curPos < this._prevPos && !this.touchDistance){
        return ;
      }
      //上划 且 有拖拽距离
      if(curPos < this._prevPos && this.touchDistance){
        this.touchDistance = Math.max(this.touchDistance + curPos - this._prevPos,0);
        let distance = ~~(this.touchDistance / this.distanceScale)
        //未达到阈值
        if(distance < this.topDistance){
          this.dragState = 'pull'
          this.$emit('top-status-change', this.dragState,distance)
        }
        //已达到阈值
        if(distance >= this.topDistance){
          this.dragState = 'drop'
          this.$emit('top-status-change', this.dragState,distance)
        }
        //设定偏移距离
        if(distance <= this.maxDistance || !this.maxDistance){
          let d = this.maxDistance ? Math.min(distance,this.maxDistance) : distance;
          setTimeout(()=>{
              this.$refs.content.style.transform = `translate3d(0,${d}px,0)`
              this.$refs.top.style.height = `${d}px`
          },0);
        }
        event.preventDefault(); 
      }
      this._prevPos = curPos;
    },
    //End
    touchEndEvent(){
      if(!this.topLoadMore){
        return;
      }
      if(this.dragState !== 'pull' && this.dragState !== 'drop'){
        return ;
      }
      if(this.dragState === 'pull'){
        setTimeout(()=>{
            this.dragState = 'none'
        },300);
        this.touchDistance = 0;
      }
      if(this.dragState === 'drop'){
        setTimeout(()=>{
            this.dragState = 'loading'
        },300);
        //将距离变更为阈值点 - 20
        this.touchDistance = (this.topDistance - 20) * this.distanceScale;
        this.topMethod && this.topMethod();
      }

      this.$emit('top-status-change', this.dragState,~~(this.touchDistance / this.distanceScale))
      this.$refs.content.style.transition = `transform 0.3s`
      this.$refs.content.style.transform = `translate3d(0,${~~(this.touchDistance / this.distanceScale)}px,0)`
      this.$refs.top.style.transition = `height 0.3s`
      this.$refs.top.style.height = `${~~(this.touchDistance / this.distanceScale)}px`
      setTimeout(()=>{
          this.$refs.content.style.transition = ``
          this.$refs.top.style.transition = ``
      },350);
    },
    onTopLoaded(){
      this.touchDistance = 0;
      this.$emit('top-status-change', this.dragState,~~(this.touchDistance / this.distanceScale))
      this.$refs.content.style.transition = `transform 0.2s`
      this.$refs.content.style.transform = `translate3d(0,${~~(this.touchDistance / this.distanceScale)}px,0)`
      this.$refs.top.style.transition = `height 0.2s`
      this.$refs.top.style.height = `0px`
      this.dragState = 'none'
      setTimeout(()=>{
          this.$refs.content.style.transition = ``
          this.$refs.top.style.transition = ``
      },250);
    }
  }
};
</script>


<style scoped>

.infinite-top-content{
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.infinite-bottom-content {
	padding: 12px 0;
}
.infinite-top-content-icon{
  display: flex;
  align-items: center;
  transition:transform 0.2s;
  
}
.icon-arrow{
  transform:rotate(180deg);
}
.icon-arrow.icon-reverse{
  transform:rotate(360deg);
}

.icon-revolve{
  animation:revolve 1.2s linear infinite;
}

@keyframes revolve
{
  from {transform:rotate(0);}
  to {transform:rotate(360deg);}
}

.infinite-top-content-title{
  font-size: 14px;
}

.infinite-top-container{
  transform:translateZ(0);
  position: relative;
  z-index: 2;
}

.infinite-list-container {
  overflow-x:hidden;
  width: 100%;
  overflow-y: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}
/* .infinite-list-container::-webkit-scrollbar{
	width: 5px;
	height: 1px;
	background-color: transparent;
} */

.infinite-list-container-no-scroll {
  overflow-x:hidden;
  width: 100%;
  overflow-y: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}
.infinite-list-container-no-scroll::-webkit-scrollbar{
	width: 0px;
	height: 1px;
	background-color: transparent;
}

.infinite-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.infinite-list {
  left: 0;
  right: 0;
  top: 0;
  z-index:1;
  position: absolute;
}

.infinite-list-item-container {
  display: flex;
}

.infinite-item{
  flex:1
}

</style>
