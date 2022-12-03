<template>
	<view class="center">
		<view class="logo" :hover-class="!sid ? 'logo-hover' : ''">
			<image class="logo-img" :src="userInfo.avatarurl ? userInfo.avatarurl :avatarurl"></image>
			<view style="flex-direction: column;margin-left: 20upx;">
				<text class="uer-name">Hi，{{sid ? userInfo.nickname : '您未登录'}}</text>
				<text class="integral">积分：{{sid ? integral : 0}}</text>
				<!-- <text class="go-sign navigat-arrow" @click="goSign">&#xe60b;</text> -->
				<!-- #ifdef H5 -->
				<!-- <text class="go-login navigat-arrow" v-if="!login" @click="goLogin">&#xe65e;</text> -->
				<!-- #endif -->
				<!-- #ifdef APP-PLUS || MP-ALIPAY || MP-TOUTIAO -->
				<!-- <text class="go-login navigat-arrow" v-if="!login" @click="goLogin">&#xe65e;</text> -->
				<!-- #endif -->
				<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
				<!-- <text class="go-login navigat-arrow" v-if="!login" @click="mpGetUserInfo">&#xe65e;</text> -->
				<!-- #endif -->
			</view>
		</view>
		<view class="center-list">
			<view class="center-list-item border-bottom">
				<text class="list-icon">&#xe60b;</text>
				<text class="list-text">签到</text>
				<text class="navigat-arrow" @click="goSign">&#xe65e;</text>
			</view>
			<!-- <view class="center-list-item border-bottom">
				<text class="list-icon">&#xe60d;</text>
				<text class="list-text">收藏图片</text>
				<text class="navigat-arrow" @click="collection_picture">&#xe65e;</text>
			</view>
			<view class="center-list-item border-bottom">
				<text class="list-icon">&#xe60c;</text>
				<text class="list-text">喜欢表情</text>
				<text class="navigat-arrow" @click="collection_images">&#xe65e;</text>
			</view> -->
		</view>
		<!-- <view class="center-list">
			<view class="center-list-item border-bottom">
				<text class="list-icon">&#xe60b;</text>
				<text class="list-text">管理图片</text>
				<text class="navigat-arrow">&#xe65e;</text>
			</view>
			<view class="center-list-item">
				<text class="list-icon">&#xe61a;</text>
				<text class="list-text">上传图片</text>
				<text class="navigat-arrow">&#xe65e;</text>
			</view>
		</view> -->
		<view class="center-list">
			<view class="center-list-item border-bottom">
				<text class="list-icon">&#xe601;</text>
				<text class="list-text">修改密码</text>
				<text class="navigat-arrow" @click="changePassword">&#xe65e;</text>
			</view>
			<!-- #ifdef MP-WEIXIN -->
			<view class="center-list-item border-bottom">
				<text class="list-icon">&#xe609;</text>
				<text class="list-text">设置账号</text>
				<text class="navigat-arrow" @click="setting">&#xe65e;</text>
			</view>
			<!-- #endif -->
			<view class="center-list-item border-bottom" @click="goAbout">
				<text class="list-icon">&#xe603;</text>
				<text class="list-text">关于我们</text>
				<text class="navigat-arrow">&#xe65e;</text>
			</view>
		</view>
		<view class="center-list" style="background-color: #EFEFEF;">
			<!-- #ifdef H5 -->
			<button type="default" @click="sid ? logout() : goLogin()" class="login" hover-class="hover">{{sid ? '退出' : '登录'}}</button>
			<!-- #endif -->
			<!-- #ifdef APP-PLUS || MP-ALIPAY || MP-TOUTIAO -->
			<button type="default" @click="sid ? logout() : goLogin()" class="login" hover-class="hover">{{sid ? '退出' : '登录'}}</button>
			<!-- <text class="go-login navigat-arrow" v-if="!login" @click="goLogin">&#xe65e;</text> -->
			<!-- #endif -->
			<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
			<button type="default" :open-type="!sid ? 'getUserInfo' : ''" @getuserinfo="mpGetUserInfo" @click="sid ? logout() : ''" class="login" hover-class="hover">{{sid ? '退出' : '登录'}}</button>
			<!-- #endif -->
			<!-- <button type="default" open-type="getUserInfo" @getuserinfo="mpGetUserInfo" class="login" hover-class="hover">{{login ? '退出' : '登录'}}</button>
			<button type="default" class="logout" hover-class="hover" formType="submit" @click="logout">退出</button> -->
		</view>
		<!-- <view :class="user_view ? 'center-list' : 'center-list user_view'" style="background-color: #EFEFEF;">
			<button type="default" @click="removeWatermark()" class="login" hover-class="hover">抖音去水印</button>
		</view> -->
	</view>
</template>

<script>
	import {
		mapState,
		mapMutations
	} from 'vuex'
	
	export default {
		data() {
			return {
				login: false,
				avatarurl: '/static/head_pic.png',
				userInfo: {},
				openid: '',
				sid: '',
				integral: 0,
				invite_uid: ''
			}
		},
		computed: {
			...mapState({
				loginProvider: state => state.loginProvider
			})
		},
		onLoad() {
			this.inviter_uid = uni.getStorageSync('inviter_uid');
			console.log(this.inviter_uid);
			
			this.sid = uni.getStorageSync('sid');
			//uni.removeStorageSync('sid');
			
			// #ifdef MP-WEIXIN
			if (!this.login) {
				uni.login({
					provider: 'weixin',
					success: (res) => {
						console.log('login success:', res);
						var code = res.code
						
						uni.request({
							url: this.$api_url+'/wx/get_openid', 
							method: 'POST',
							header: {'content-type': 'application/x-www-form-urlencoded'},
							data: {code: code},
							success: (res) => {
								this.openid = res.data.data.openid;
								uni.setStorage({
									key: 'openid',
									data: this.openid,
									success: function () {
										console.log('success');
									}
								});
							},
							fail: (err) => {
								console.log('获取openid失败:', err);
							}
						});
					},
					fail: (err) => {
						console.log('login fail:', err);
					}
				});
			}
			// #endif
			
			this.get_UserInfo();
			this.get_Integral();
			
			setTimeout(function () {
				uni.startPullDownRefresh();
				console.log('start pulldown');
			}, 1000);
		},
		methods: {
			get_Integral() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					return;
				}
				var uid = uni.getStorageSync('uid');
				uni.request({
					url: this.$api_url + '/user/get_integral',
					method: 'POST',
					header: {'content-type': 'application/x-www-form-urlencoded'},
					data: {uid:uid},
					success: (result) => {
						if (result.statusCode !== 200) {
							return;
						}
						this.integral = result.data.data.integral;
						
						console.log(result.data.data);
					},
					fail: (err) => {
						console.log('获取接口返回错误:', err);
					},
				});
			},
			get_UserInfo() {
				var sid = uni.getStorageSync('sid');
				//uni.removeStorageSync('sid');
				
				uni.request({
					url: this.$api_url+'/user/get_login', 
					method: 'POST',
					header: {'content-type': 'application/x-www-form-urlencoded'},
					data: {sid:sid},
					success: (res) => {
						console.log(res);
						if(res.data.code==200) {
							this.login = true;
							this.userInfo = res.data.data;
							this.userInfo.nickname = this.userInfo.nickname?this.userInfo.nickname:this.userInfo.username;
						} else {
							this.login = false;
							console.log('登录失败:', res.data.msg);
						}
					},
					fail: (err) => {
						this.login = false;
						console.log('登录失败:', err);
					}
				});
			},
			
			logout() {
				if (!this.login) {
					uni.showModal({
						title: '退出失败',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				uni.showModal({
					content: '您确定退出么？',
					success: (res) => {
						if (res.confirm) {
							
							var sid = uni.getStorageSync('sid');
							uni.request({
								url: this.$api_url+'/user/logout', 
								method: 'POST',
								header: {'content-type': 'application/x-www-form-urlencoded'},
								data: {sid:sid},
								success: (res) => {
									console.log(res);
									if(res.data.code==200) {
										this.login = false;
										uni.removeStorage({
										    key: 'sid',
										    success: function (res) {
										        console.log('success');
										    }
										});
										uni.removeStorageSync('uid');
										uni.removeStorageSync('is_member');
										uni.removeStorageSync('openid');
										this.userInfo = {};
										this.sid = uni.getStorageSync('sid');
										
										uni.showModal({
											title: '退出成功',
											content: '欢迎下次再来！',
											showCancel: false
										});
										
									} else {
										console.log('退出失败:', res.data.msg);
									}
								},
								fail: (err) => {
									console.log('退出失败:', err);
								}
							});
							
							/* uni.navigateTo({
								url: '/pages/user/index'
							}); */
						}
					}
				})
			},
			setting() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				uni.navigateTo({
					url: '/pages/user/setting?query=' + encodeURIComponent(JSON.stringify(this.userInfo))
				});
			},
			changePassword() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				uni.navigateTo({
					url: '/pages/user/password'
				});
			},
			goLogin() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.navigateTo({
						url: '/pages/login/login'
					});
				}
			},
			goSign() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				
				uni.navigateTo({
					url: '/pages/user/sign'
				});
			},
			//...mapMutations(['login']),
			getUserInfo() {	
				if (!this.login) {
					uni.login({
						provider: 'weixin',
						success: (res) => {
							console.log('login success:', res);
							// 更新保存在 store 中的登录状态
							this.login('weixin');
						},
						fail: (err) => {
							console.log('login fail:', err);
						}
					});
				}
				uni.getUserInfo({
					provider: this.loginProvider,
					success: (result) => {
						console.log('getUserInfo success', result);
						this.login = true;
						this.userInfo = result.userInfo;
						uni.showModal({
							title: '登录成功',
							content: '欢迎来到文字一键转成语音中心！',
							showCancel: false
						});
						uni.navigateTo({
							url: '/pages/user/index'
						});
					},
					fail: (error) => {
						console.log('getUserInfo fail', error);
						let content = error.errMsg;
						//if (~content.indexOf('uni.login')) {
							//content = '请在登录页面完成登录操作';
						//}
						uni.getSetting({
							success: (res) => {
								let authStatus = res.authSetting['scope.userInfo'];
								if (!authStatus) {
									uni.showModal({
										title: '授权失败',
										content: 'Hello uni-app需要获取您的用户信息，请在设置界面打开相关权限',
										success: (res) => {
											if (res.confirm) {
												uni.openSetting()
											}
										}
									})
								} else {
									uni.showModal({
										title: '获取用户信息失败',
										content: '错误原因' + content,
										showCancel: false
									});
								}
							}
						})
					}
				});
			},
			// #ifdef MP-WEIXIN
			mpGetUserInfo(result) {
				if (!this.openid) {
					uni.showModal({
						title: '登录失败!',
						content: '请刷新页面再登录!',
						showCancel: false
					});
					return;
				}
				
				this.login = true;
				this.userInfo = result.detail.userInfo;
				this.userInfo.openid = this.openid;
				this.userInfo.inviter_uid = this.inviter_uid;
				
				uni.request({
					url: this.$api_url+'/user/save',
					method: 'POST',
					header: {'content-type': 'application/x-www-form-urlencoded'},
					data: this.userInfo,
					success: (res) => {
						uni.showModal({
							title: '登录成功',
							content: '欢迎来到文字一键转成语音中心！',
							showCancel: false
						});
						/* uni.navigateTo({
							url: '/pages/user/index'
						}); */
						
						this.userInfo = res.data.data;
						this.sid = this.userInfo.sid;
						var uid = this.userInfo.uid;
						var is_member = this.userInfo.is_member;
						
						uni.setStorageSync('sid', this.sid);
						uni.setStorageSync('uid', uid);
						uni.setStorageSync('is_member', is_member);
						uni.setStorageSync('openid', this.openid);
						
						this.get_Integral();
						
						console.log(res)
					},
					fail: (err) => {
						uni.showModal({
							title: '登录失败',
							content: '请重新登录！',
							showCancel: false
						});
						console.log('保存用户信息失败:', err);
					}
				});
			},
			// #endif
			goAbout() {
				uni.navigateTo({
					url: '/pages/about/index'
				});
			},
			removeWatermark() {
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				uni.navigateTo({
					url: '/pages/video/index'
				});
			},
			onPullDownRefresh() {
				// #ifndef MP-WEIXIN
				this.get_UserInfo();
				// #endif
				this.get_Integral();
				setTimeout(function () {
					uni.stopPullDownRefresh();
				}, 1000);
			}
		}
	}
</script>

<style>
	page,
	view {
		display: flex;
	}
	
	.user_view {
		display: none;
	}
	
	.integral {
		color: #FFFFFF;
	}
	
	button.login {
		background-color: #228FCD;
		color: #fff;
	}
	button.logout {
		background-color: #228FCD;
		color: #fff;
	}
</style>
