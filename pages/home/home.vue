<template>
	<view>
		<!-- 搜索 -->
		<view class="search-box">
			<my-search @click="gotoSearch"></my-search>
		</view>
		<!-- 轮播图区域 -->
		<swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
			<swiper-item v-for="(item,index) in swiperList" :key="index">
				<navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id='+item.goods_id">
					<image :src="item.image_src" ></image>
				</navigator>
			</swiper-item>
		</swiper>
		<!-- 分类导航区 -->
		<view class="nav-list">
			<view class="nav_item" v-for="(item,i) in navList" :key="i" @click="navClickHandler(item)">
				<image :src="item.image_src" class="nav-img"></image>
			</view>
		</view>
		<!-- 楼层的数据 -->
		<view class="floor-list">
			<!-- 楼层item项 -->
			<view class="floor-item" v-for="(item , i) in floorList" :key="i">
				<image :src="item.floor_title.image_src" class="floor-title"></image>
				<!-- 楼层图片区域 -->
				<view class="floor-img-box">
				  <!-- 左侧大图片的盒子 -->
				  <navigator class="left-img-box" :url="item.product_list[0].url">
					  <!-- 因为左侧大图片是这个数组的第0项 且后端提供了宽度数据 -->
				    <image :src="item.product_list[0].image_src" :style="{width: item.product_list[0].image_width + 'rpx'}" mode="widthFix"></image>
				 </navigator>
				  <!-- 右侧 4 个小图片的盒子 -->
				  <view class="right-img-box">
					  <!-- 删掉第0项，模式用widthfix -->
				    <navigator class="right-img-item" v-for="(item2, i2) in item.product_list" :key="i2" v-if="i2 !== 0" :url="item2.product_list.url">
				      <image :src="item2.image_src" mode="widthFix" :style="{width: item2.image_width + 'rpx'}"></image>
				    </navigator>
				  </view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
// 导入自己封装的 mixin 模块
	import badgeMix from '@/mixins/tabbar-badge.js'
	export default {
		 // 将 badgeMix 混入到当前的页面中进行使用
		mixins: [badgeMix],
		data() {
			return {
				//轮播图数据列表
				swiperList:[],
				//分类导航数据列表
				navList:[],
				//获取楼层数据
				floorList: []
			};
		},
		onLoad(){
			this.getSwiperList()
			this.getNavList()
			this.getFloorList()
		},
		methods:{
			async getSwiperList(){
				const {data : res}=await uni.$http.get('/api/public/v1/home/swiperdata')
				if(res.meta.status != 200) return uni.$showMsg()
				this.swiperList = res.message
			},
			async getNavList(){
				const {data : res} = await uni.$http.get('/api/public/v1/home/catitems')
				if(res.meta.status != 200) return uni.$showMsg()
				this.navList = res.message
			},
			navClickHandler(item){
				 if (item.name === '分类') {
				    uni.switchTab({
				      url: '/pages/cate/cate'
				    })
				  }
			},
			async getFloorList() {
			      const { data: res } = await uni.$http.get('/api/public/v1/home/floordata')
			      if (res.meta.status !== 200) return uni.$showMsg()
				  //先对数据进行处理
				  res.message.forEach(floor => {
					  floor.product_list.forEach(prod => {
						  prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]
					  })
				  })
			      this.floorList = res.message
			    },
				gotoSearch() {
				  uni.navigateTo({
				    url: '/subpkg/search/search'
				  })
				}
		},
	}
</script>

<style lang="scss">
   swiper{
	   height: 330rpx;
	   .swiper-item,
	   image{
		   width: 100%;
		   height: 100%;
	   }
   }
   .nav-list{
	   display:flex;
	   justify-content: space-around;
	   margin:15px 0; 
	   .nav-img{
	   		   width:128rpx;
	   		   height: 140rpx;
	   }
   }
   .floor-title {
     height: 60rpx;
     width: 100%;
     display: flex;
   }
   .right-img-box {
     display: flex;
     flex-wrap: wrap;//横向布局
     justify-content: space-around;//分散布局
   }
   
   .floor-img-box {
     display: flex;//确保两个盒子是左右的
     padding-left: 10rpx;
   }
   .search-box {
     // 设置定位效果为“吸顶”
     position: sticky;
     // 吸顶的“位置”
     top: 0;
     // 提高层级，防止被轮播图覆盖
     z-index: 999;
   }
</style>
