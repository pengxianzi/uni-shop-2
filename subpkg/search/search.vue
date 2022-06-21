<template>
	<view>
		<view class="search-box" :style="{'background-color':'#c00000'}">
			<uni-search-bar placeholder="请输入搜索内容"  @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		</view>
	
	<!-- 搜索建议列表 -->
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item,i) in searchResults" :key="i" @click="gotoDetail(item)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
		
		
	<!-- 搜索历史 -->
	
		<view class="history-box" v-else>
			<!-- 标题区域 -->
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="clean"></uni-icons>
			</view>
			<!-- 列表区域 -->
			<view class="history-list">
				<uni-tag :text="item" v-for="(item,i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timer:null,
				kw:'',
				//搜索的结果列表
				searchResults:[],
				//搜索历史的数组
				historyList:[]
			};
		},
		onLoad() {
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
		},
		methods:{
			// input输入事件处理函数
			input(e){
				// console.log(e)
				
				
				clearTimeout(this.timer)
				
				this.timer = setTimeout(() => {
					// console.log(e)
					this.kw = e
					this.getSearchList()
				},500)
			},
			async getSearchList(){
				//判断搜索关键词是否为空
				if(this.kw.length === 0){
					this.searchResults=[]
					return
				}
				
				const {data:res} = await uni.$http.get('/api/public/v1/goods/qsearch', { query:this.kw })
				if(res.meta.status !== 200) return uni.$showMes()
				this.searchResults = res.message
				
				
				this.saveSearchHistory()
			},
			gotoDetail(item){
				 // console.log(item.goods_id)
				 uni.navigateTo({
				 	url:'/subpkg/goods_detail/goods_detail?goods_id' + item.goods_id
				 })
			},
			saveSearchHistory(){
				// this.historyList.push(this.kw)
				
				 // 1. 将 Array 数组转化为 Set 对象
				const set = new Set(this.historyList)
				 // 2. 调用 Set 对象的 delete 方法，移除对应的元素
				set.delete(this.kw)
				 // 3. 调用 Set 对象的 add 方法，向 Set 中添加元素
				set.add(this.kw)
				 // 4. 将 Set 对象转化为 Array 数组
				this.historyList = Array.from(set)
				
				// console.log(set)
				
				
				// 对搜索历史数据,进行持久化的储存
				uni.setStorageSync('kw',JSON.stringify(this.historyList))
			},
			clean(){
				this.historyList = []
				uni.setStorageSync('kw','[]')
			},
			gotoGoodsList(kw){
				uni.navigateTo({
					url:'/subpkg/goods_list/goods_list?query=' + kw
				})
			}
		},
		computed:{
			histories(){
				return [...this.historyList].reverse()
			}
		}
	}
</script>

<style lang="scss">
	.search-box{
		position: sticky;
		top: 0;
		z-index: 999;
	}
	
	.sugg-list{
		padding: 0 5px;
		
		.sugg-item{
			display: flex;
			align-items: center;
			justify-content: space-between;
			font-size: 12px;
			padding: 13px 0;
			border-bottom: 1px solid #efefef;
			.goods-name{
				  // 文字不允许换行（单行文本）
				white-space: nowrap;
				 // 溢出部分隐藏
				overflow: hidden;
				 // 文本溢出后，使用 ... 代替
				text-overflow: ellipsis;
				margin-right: 3px;
			}
		}
	}
	
	
	.history-box{
		padding: 0 5px;
		
		.history-title{
			display: flex;
			justify-content: space-between;
			height: 40px;
			align-items: center;
			font-size: 13px;
			border-bottom: 1px solid #efefef;
		}
		.history-list{
			display: flex;
			flex-wrap: wrap;
			
			.uni-tag{
				margin-top: 5px;
				margin-right: 5px;
				background-color: #efefef;
				color: black;
				border: 0;
			}
		}
	}
</style>
