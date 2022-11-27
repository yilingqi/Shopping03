<template>
	<view>
		<my-search @mySearch="gotoSearch"></my-search>
	<!-- scoll-view 纵向滑动要提供一个固定的高度和scoll-y-->
	<view class="scroll-view-container">
		<!-- 左侧滑动区 -->
		<scroll-view class="left-scroll-view" scroll-y="true" :style="{height:wh+'px'}">
			<!-- block相当于template -->
			<block v-for="(item , i) in cateList" :key="i">
				<!-- 如果索引值等于active值，那么就绑定active样式 -->
				<!-- 绑定一个数组，left-scroll-view-item是必须有的，active是按需设置的 -->
				<view :class="['left-scroll-view-item',i == active? 'active':'']" @click="activeChanged(i)">{{item.cat_name}}</view>
			</block>
		</scroll-view>
		<!-- 右侧滑动区 -->
		<scroll-view class="right-scroll-view" scroll-y="true" :style="{height:wh+'px'}" :scroll-top="scrollTop">
			<!-- :scroll-top="scrollTop"动态绑定scrollTop使切换页面的时候，滚动条自动回到顶部 -->
			<view class="cate-lv2" v-for="(item2, i2) in cateLevel2" :key="i2">
			<view class="cate-lv2-title">
				/{{item2.cat_name}}/
			</view>
			 <view class="cate-lv3-list">
			       <!-- 三级分类 Item 项 -->
			       <view class="cate-lv3-item" v-for="(item3, i3) in item2.children" :key="i3" @click="gotoGoodsList(item3)">
			         <!-- 图片 -->
			         <image :src="item3.cat_icon"></image>
			         <!-- 文本 -->
			         <text>{{item3.cat_name}}</text>
			       </view>
			     </view>
			</view>
		</scroll-view>
	</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wh:0,//可使用屏幕的高度
				scrollTop:0,//滚动条
				cateList:[],//一级分类
				active:0,
				cateLevel2:[],//二级分类
			};
		},
		onLoad(){
			//获取手机设备信息，里面有一个可使用屏幕的高度
			const sysInfo = uni.getSystemInfoSync()
			this.wh = sysInfo.windowHeight-50
			//后来在页面上加了一个搜索框，需要减掉搜索框的高度
			this.getCateList()
		},
		methods:{
			async getCateList(){
				const {data : res} = await uni.$http.get('/api/public/v1/categories')
				if (res.meta.status !== 200) return uni.$showMsg()
				this.cateList = res.message
				 this.cateLevel2 = res.message[0].children//为二级分类赋初始值
			},
			activeChanged(i){
				this.active = i
				//重新为二级分类赋值渲染
				this.cateLevel2 = this.cateList[i].children
				this.scrollTop = this.scrollTop ? 0 : 1
			},
			gotoGoodsList(item3) {
			  uni.navigateTo({
			    url: '/subpkg/goods_list/goods_list?cid=' + item3.cat_id
			  })
			},
			gotoSearch(){
				uni.navigateTo({
					url:'/subpkg/search/search'
				})
			}
		}
	}
</script>

<style lang="scss">
.scroll-view-container{
	display:flex;
	.left-scroll-view{
		width:120px;
	}
	.right-scroll-view{
		background-color: #fff;
	}
	.left-scroll-view-item{
		background-color: #f7f7f7;
		line-height: 60px;
		text-align: center;
		font-size: 12px;
		
		&.active{
			background-color: #fff;
			position: relative;
			&::before{
				content:"";
				display: block;
				width: 3px;
				height: 30px;
				background-color: #c00000;
				position: absolute;
				top:50%;
				left: 0;
				transform: translateY(-50%);
			}
		}
	}
}
.cate-lv2-title {
  font-size: 12px;
  font-weight: bold;
  text-align: center;
  padding: 15px 0;
}
.cate-lv3-list{
	display:flex;//弹性布局
	flex-wrap: wrap;//自动换行
	.cate-lv3-item{
		width: 33.33%;//一行有3个
		display: flex;
		flex-direction: column;//让文字和图片纵向对齐
		justify-content: center;//纵向居中
		align-items: center;
		margin-bottom:10px;
		image{
			width: 60px;
			height: 60px;
		}
		text{
			font-size: 12px;
		}
	}
}
</style>
