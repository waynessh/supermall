<template>
  <div class="wrapper" ref="wrapper">
    <div class="content">
      <slot></slot>
    </div>
  </div>
</template>

<script>
import BScroll from 'better-scroll'
export default {
  name:"Scroll",
  props:{
    probeType:{
      type:Number,
      default:0
    },
    PullUpLoad:{
      type: Boolean,
      default: true
    },
  },
  data(){
    return{
      scroll:null,
    }
  },
  mounted(){
    // 1.创建BScroll对象
    // console.log(this.$refs.wrapper);
    this.scroll = new BScroll(this.$refs.wrapper,{
      observeDOM: true,
      click: true,
      //  probeType:3,
      probeType: this.probeType,
      pullUpLoad: this.PullUpLoad
    })
    // 2.监听滚动的位置
  //  if(this.probeType === 2 || this.probeType === 3){
    
  //  }
    this.scroll.on('scroll',(position) =>{
      // console.log(position);
      this.$emit('scroll',position)
    })

    // 3.监听上拉下载更多
     if(this.PullUpLoad){
        this.scroll.on('pullingUp',() =>{
    // console.log('上拉加载更多');
    this.$emit('pullingUp')
  })
    }
 
  },
  methods:{
    scrollTo(x,y,time=300){
      this.scroll && this.scroll.scrollTo(x,y,time)
    },
    finishPullUp(){
     this.scroll &&  this.scroll.finishPullUp()
    },
    refresh(){
    this.scroll && this.scroll.refresh()
    },
    getScrollY(){
      // 三步运算符
      return this.scroll ? this.scroll.y :0
    }
  }
}
</script>

<style>

</style>