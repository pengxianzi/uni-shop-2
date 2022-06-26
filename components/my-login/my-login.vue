<template>
	<view class="login-container">
		<uni-icons type="contact-filled" size="100" color="#afafaf"></uni-icons>
		<button type="primary" class="btn-login" @click="getUserProfile">一键登录</button>
		<text class="tips-text">登录后尽享更多权益</text>
	</view>
</template>

<script>
	import { mapMutations,mapState } from 'vuex'
	export default {
		name:"my-login",
		data() {
			return {
				
			};
		},
		computed:{
			...mapState('m_user',['redirectInfo'])
		},
		methods:{
			...mapMutations('m_user',['updateUserInfo','updateToken','updateRedirectInfo']),
			// 用户授权之后,获取用户的基本信息
			getUserProfile(e){
			            uni.getUserProfile({
			              desc: '用于完善个人资料', // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
			              success: (res) => {
			                // 3. 将用户的基本信息存储到 vuex 中
			              this.updateUserInfo(res.userInfo)
			              
			              // 获取登录成功后的 Token 字符串
			              this.getToken(res)
			              },
			               fail: (res) => {
			                    return uni.$showMsg('您取消了登录授权')
			                  }
			            })
					},
			async getToken(info){
				// 获取code对应的值
				const [err,res] = await uni.login().catch(err => err)
				// console.log(res);
				
				if(err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')
				
				
				// console.log(res.code);
				// console.log(info);
				
				
				// 准备参数
				const query = {
					code: res.code,
					encryptedData: info.encryptedData,
					iv: info.iv,
					rawData: info.rawData,
					signature: info.signature
				}
				
				
				// console.log(query);
				
				const {data : loginResult} = await uni.$http.post('/api/public/v1/users/wxlogin',query)
				if(loginResult.meta.state == 200) return uni.$showMsg('登录失败！')
				// uni.$showMsg('登录成功！')
				
				// 直接把 token 保存到 vuex 中
				this.updateToken("loginResult.message.token");
				uni.$showMsg("由于登录接口 400 只能自己写 taoken")
				
				this.navigateBack()
			},
			navigateBack(){
				if(this.redirectInfo && this.redirectInfo.openTtpe === 'switchTab'){
					uni.switchTab({
						url: this.redirectInfo.from,
						complete: () => {
							this.updateRedirectInfo(null)
						}
					})
				}
			}
		}
	}
</script>

<style lang="scss">


	.login-container{
		height: 750rpx;
		background-color: #f8f8f8;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: relative;
		overflow: hidden;
		
		&::after{
			content: '';
			display: block;
			width: 100%;
			height: 40px;
			background-color: white;
			position: absolute;
			bottom: 0;
			left: 0;
			border-radius: 100px;
			transform: translateY(50%);
		}
		
		
		.btn-login{
			width: 90%;
			border-radius: 100px;
			margin: 15px 0;
			background-color: #c00000;
			
		}
		.tips-text{
			font-size: 12px;
			color: gray;
		}
	}
</style>