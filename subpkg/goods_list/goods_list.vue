<template>
  <view class="goods-page">
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i" @click="gotoDetail(goods)">
		  <my-goods :goods='goods'></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
	export default {
		data() {
			return {
				//请求参数对象
				queryObj:{
					// 查询关键词
					      query: '',
					      // 商品分类Id
					      cid: '',
					      // 页码值
					      pagenum: 1,
					      // 每页显示多少条数据
					      pagesize: 10
				},
				goodsList:[],//商品列表
				total:'',//用来分页
				isLoading:false//节流阀
			};
		},
		onLoad(options) {
			//options里面有传过来的参数
		  // 将页面参数转存到 this.queryObj 对象中
		  this.queryObj.query = options.query || ''
		  this.queryObj.cid = options.cid || ''
		  this.getGoodsList()
		},
		methods:{
			//在这里接受一个cb，可能会传这个cb，可能不会传
			async getGoodsList(cb){
				//打开节流阀
				this.isLoading = true
				cb && cb() //如果cb存在，那么就调用cb函数，如果不存在，就不调用 
				
				const {data : res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
				if(res.meta.status != 200) return uni.$showMsg()
				//拼接旧数据与新数据
				this.goodsList = [...this.goodsList, ...res.message.goods]
				this.total = res.message.total
				//关闭节流阀
				this.isLoading = false
			},
			gotoDetail(item){
				uni.navigateTo({
					 url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			}
		},
		onReachBottom() {
			//看数据是否加载完毕
			if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')
			if(this.isLoading) return
		  // 让页码值自增 +1
		  this.queryObj.pagenum += 1
		  // 重新获取列表数据
		  this.getGoodsList()
		},//先打开项目根目录中的 pages.json 配置文件，为 subPackages 分包中的 goods_list 页面配置上拉触底的距离："onReachBottomDistance": 150
		// 下拉刷新的事件
		//在 pages.json 配置文件中，为当前的 goods_list 页面单独开启下拉刷新效果："enablePullDownRefresh": true, "backgroundColor": "#F8F8F8"
		onPullDownRefresh() {
		  // 1. 重置关键数据
		  this.queryObj.pagenum = 1
		  this.total = 0
		  this.isloading = false
		  this.goodsList = []
		  // 2. 重新发起请求,传一个回调函数，当我们数据加载完毕的时候，要关闭刷新效果
		  this.getGoodsList(() => uni.stopPullDownRefresh())
		  //cb使callback 这里有一个callback函数
		}
	}
</script>

<style lang="scss">
.goods-page{
	background-color: #fff;
}
</style>
