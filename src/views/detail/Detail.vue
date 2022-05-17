<template>
  <div class="detail">
     <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav-bar>
  <scroll class="content" ref="scroll" @scroll="contentScroll" :probeType="3">
    <!-- <ul>
      <li v-for="(item,index) in $store.state.cartList" :key="index">{{item}}</li>
    </ul> -->
  <detail-swiper :topImages="topImages"></detail-swiper>
  <detail-base-info :goods="goods"></detail-base-info>
  <detail-shop-info :shop="shop"></detail-shop-info>
  <detail-goods-info :detailInfo="detailInfo" @image-load="imageload"></detail-goods-info>
  <detail-param-info ref="params" :paramInfo="paramInfo"></detail-param-info>
  <detail-comment-info ref="comment"  :commentInfo="commentInfo"></detail-comment-info>
  <goods-list ref="recommend" :goods="recommends"></goods-list>
  </scroll>
  <detail-bottom-bar @addCart="addToCart"></detail-bottom-bar>
  <back-top @click.native="backTop" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar.vue'
import DetailSwiper from './childComps/DetailSwiper.vue'
import DetailBaseInfo from './childComps/DetailBaseInfo.vue'
import DetailShopInfo from './childComps/DetailShopInfo.vue'
import Scroll from '../../components/common/scroll/Scroll.vue'
import DetailGoodsInfo from './childComps/DetailGoodsInfo.vue'
import DetailParamInfo from './childComps/DetailParamInfo.vue'
import DetailCommentInfo from './childComps/DetailCommentInfo.vue'
import GoodsList from '../../components/content/goods/GoodsList.vue'
import DetailBottomBar from './childComps/DetailBottomBar.vue'
import BackTop from '../../components/content/backTop/BackTop.vue'

import {getDetail,Goods,Shop,GoodsParam,getRecommend} from '../../network/detail'
import {debounce} from '../../common/utils'
export default {
name:"Detail",
components:{
  DetailNavBar,
  DetailSwiper,
  DetailBaseInfo,
  DetailShopInfo,
  Scroll,
  DetailGoodsInfo,
  DetailParamInfo,
  DetailCommentInfo,
  GoodsList,
  DetailBottomBar,
  BackTop
},
data(){
  return{
    iid:null,
    topImages:[],
    goods:{},
    shop:{},
    detailInfo:{},
    paramInfo:{},
    commentInfo:{},
    recommends:[],
    themeTopYs:[],
    getThemeTopY:null,
    currentIndex:0,
    isShowBackTop:false
  }
},
created(){
  // 1.保存传入的iid
  this.iid = this.$route.params.iid

  //2.根据iid请求详情数据
  getDetail(this.iid).then(res =>{
    // console.log(res);
    // 1.获取顶部的图片轮播数据
    const data = res.result
    this.topImages = data.itemInfo.topImages

    //2.获取商品信息
    this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
    //3.创建店铺信息的对象
    this.shop = new Shop(data.shopInfo)

    //4.保存商品的详情数据
    this.detailInfo = data.detailInfo;

    //5.获取参数信息
    this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule) 

    //6.取出参数信息
    // this.itemParams = data.itemParams

    // 7.取出评论信息
    if(data.rate.cRate !== 0){
      this.commentInfo = data.rate.list[0]
    }
  //   this.$nextTick(() =>{
  //     //根据最新数据，对应的DOM是已经被渲染出来
  //     //但是图片仍然没有加载完(目前获取到offsetTop不包含其中的图片)
  //     //offsetTop值不对的时候，就是应为图片的问题
  //     this.themeTopYs = []
  // this.themeTopYs.push(0)
  // this.themeTopYs.push(this.$refs.params.$el.offsetTop)
  // this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
  // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
  // console.log(this.themeTopYs);
  //   })
  })
  
  //3.请求推荐数据
  getRecommend().then(res =>{
    // console.log(res);
    this.recommends = res.data.list
  })
   //4.给getThemeTopY赋值
   this.getThemeTopY = setTimeout(() =>{
     this.themeTopYs = []
     this.themeTopYs.push(0);
     this.themeTopYs.push(this.$refs.params.$el.offsetTop)
     this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
     this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

  // console.log(this.themeTopYs);
  },500)
  
},
methods:{
  imageload(){
    this.$refs.scroll.refresh()
    this.getThemeTopY()
    
  },
  titleClick(index){
    // console.log(index);
    this.$refs.scroll.scrollTo(0, -this.themeTopYs[index]+44,100)
  },
  contentScroll(position){
    // 1.获取y值
    // console.log(position);
    const positionY = -position.y
    // 2.positionY和主题中值进行对比
    let length = this.themeTopYs.length
    for(let i=0;i<length;i++){
       if(this.currentIndex !== i && ((i <length -1 && positionY >= this.themeTopYs[i] &&
        positionY < this.themeTopYs[i+1]) || (i === length -1 && positionY >= this.themeTopYs[i]))){
         this.currentIndex = i
        //  console.log(this.currentIndex);
        this.$refs.nav.currentIndex = this.currentIndex
       }
    }
    //3.是否显示回到顶部
        this.isShowBackTop = (-position.y) > 1000;
  },
    backTop(){
    this.$refs.scroll.scrollTo(0,0)
  },
  addToCart(){
    // console.log('addToCart');
    //1.获取购物车需要展示的信息
    const product = {}
    product.image = this.topImages[0]
    product.title = this.goods.title;
    product.desc = this.goods.desc;
    product.price = this.goods.newPrice
    product.iid = this.iid
    product.realPrice = this.goods.realPrice

    //2.将商品添加到购物车里
    // this.$store.cartList.push(product)
    // this.$store.commit('addCart',product)
    this.$store.dispatch('addCart',product)
  }
}

}
</script>

<style scoped>
.detail{
  position: relative;
  /* position: absolute; */
  z-index: 9;
  background-color: #fff;
  height: 100vh;
}
.content{
  height: calc(100% - 44px);
  overflow: hidden;
}
.detail-nav{
  position: relative;
  z-index: 9;
  background-color: #fff;
}
</style>