<template>
 <div id="home" class="wrapper">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
     <tab-control :titles="['流行','新款','精选']" 
     @tabClick="tabClick" 
     ref="tabControl1" 
     class="tab-control" v-show="isTabFixed"></tab-control>
    <scroll class="content" ref="scroll" :probe-type="3"  @scroll="contentScroll" :pull-up-load="true" @pullingUp="loadMore">
     
    <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
    <recommend-view :recommends="recommends"></recommend-view>
    <feature-view></feature-view>
    <tab-control :titles="['流行','新款','精选']" 
    @tabClick="tabClick" 
    ref="tabControl2"></tab-control>
    <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <!-- 修饰符.native属性可以监听不是原生标签 -->
     <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
     </div>
</template>

<script>
import NavBar from '../../components/common/navbar/NavBar.vue'
import HomeSwiper from './childComps/HomeSwiper.vue'
import RecommendView from '../home/childComps/RecommendView.vue'

import FeatureView from '../home/childComps/FeatureView.vue'
import TabControl from '../../components/content/tabControl/TabControl.vue'
import GoodsList from '../../components/content/goods/GoodsList.vue'
import Scroll from '../../components/common/scroll/Scroll.vue'
import BackTop from '../../components/content/backTop/BackTop.vue'

import {getHomeMultidata,getHomeGoods} from '../../network/home'
export default {
name:"Home",
components:{
  NavBar,
  HomeSwiper,
  RecommendView,
  FeatureView,
  TabControl,
   GoodsList,
   Scroll,
   BackTop
},
data(){
  return{
    //对getHomeMultidata函数输出的数据进行保存
    // result:null
    banners:[],
    recommends:[],
    goods:{
      'pop':{page:0,list:[]},
      'new':{page:0,list:[]},
      'sell':{page:0,list:[]},
    },
    currentType:'pop',
    isShowBackTop:false,
    // pullupload:true
    tabOffsetTop:0,
    isTabFixed:false,
    saveY:0,
  }
},
computed:{
  showGoods(){
    return this.goods[this.currentType].list
  }
},
activated(){
  // console.log('activated');
  this.$refs.scroll.refresh()
  this.$refs.scroll.scrollTo(0,this.saveY,0)
  
},
deactivated(){
  //1.保存y值
  // console.log(this.saveY);
  this.saveY = this.$refs.scroll.getScrollY()

},
//vue的生命周期，在首页created时，请求轮播图数据
created(){
   //1.请求多个数据
this.getHomeMultidata()

  // 2.请求商品数据
  this.getHomeGoods('pop')
  this.getHomeGoods('new')
  this.getHomeGoods('sell')

},
mounted(){
  //1.图片加载完成时的事件监听
  const refresh = this.debounce(this.$refs.scroll.refresh)

  // 3.监听item中图片加载完成
  this.$bus.$on('itemImageLoad',() =>{
    refresh()
  })
  // 2.获取tabControl的offsetTop
  // this.tabOffsetTop = this.$refs.offsetTop

},
methods:{

  // 事件监听相关方法，防抖
  debounce(func,delay){
    let timer = null
    return function(...args){
     if(timer) clearTimeout(timer)
    timer = setTimeout(() =>{
      func.apply(this,args)
    },delay)
    }
  },
  //事件监听相关方法
  tabClick(index){
    // console.log(index);
    switch(index){
      case 0:
        this.currentType = 'pop'
        break
        case 1:
         this.currentType ='new'
         break
         case 2:
         this.currentType = 'sell'
         break
    }
    this.$refs.tabControl1.currentIndex = index
     this.$refs.tabControl2.currentIndex = index
  },
  backClick(){
    this.$refs.scroll.scrollTo(0,0)
  },
  contentScroll(position){
    // 1.判断BackTop是否显示
   this.isShowBackTop =  (-position.y > 1000 )
  //  2.决定tabControl是否吸顶(position：fixed)
  this.isTabFixed = (-position.y) > this.tabOffsetTop
  },
  loadMore(){
    // console.log('上拉');
    this.getHomeGoods(this.currentType)
    this.$refs.scroll.refresh()
  },
  swiperImageLoad(){
    // console.log(this.$refs.tabControl.$el.offsetTop);
    this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
  },
  //网络请求相关方法
  getHomeMultidata(){
  getHomeMultidata().then(res => {
    // this.result = res;
    this.banners = res.data.banner.list;
    this.recommends = res.data.recommend.list;
  })
  },
   getHomeGoods(type){
     const page = this.goods[type].page + 1
  getHomeGoods(type,page).then(res =>{
    // console.log(res);
    this.goods[type].list.push(...res.data.list)
    this.goods[type].page += 1
    
    //完成上拉加载更多
    this.$refs.scroll.finishPullUp()
  })
  },
}
}
</script>

<style scoped>
#home{
  height: 100vh;
  position: relative;
}
.home-nav{
  background-color: var(--color-tint);
  /* overflow: hidden; */
  /* z-index: 50; */
  /* position: fixed;
  top: 0px;
  left: 0;
  width: 100%;
  text-align: center; */
  color: #fff;
}
.tab-control{
  position: relative;
  /* top: 44px; */
  z-index: 9;
}
.content{
  position: absolute;
  overflow: hidden;
  top: 44px;
  bottom: 49px;
}
/* .fixed{
  position: fixed;
  right: 0;
  left: 0;
  top: 44px;
} */
</style>