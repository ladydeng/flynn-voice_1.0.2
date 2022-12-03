<template>
	<form class='loginView' @submit="login">
		<view class="input-view">
			<view class="label-view" style="width:90px;">
				<text class="label">账号 </text>
			</view>
			<input class="input" type="text" @input="onUsername" placeholder="请输入账号" name="username" :value="userinfo.username"/>
		</view>
		<view class="input-view">
			<view class="label-view" style="width:90px;">
				<text class="label">密码 </text>
			</view>
			<input class="input" type="password" @input="onPassword" placeholder="请输入登录密码" name="password" :value="userinfo.password"/>
		</view>
		<view class="button-view">
			<button type="default" class="login" hover-class="hover" @click="setting" style="background-color: #228FCD;">绑定</button>
		</view>
	</form>
</template>

<script>
	export default {
		data() {
			return {
				userinfo: {},
				uid: '',
				username: '',
				password: '',
			};
		},
		onLoad(event) {
			// 目前在某些平台参数会被主动 decode，暂时这样处理。
			try {
				this.userinfo = JSON.parse(decodeURIComponent(event.query));
			} catch (error) {
				this.userinfo = JSON.parse(event.query);
			}
			
		},
		methods: {
			onUsername: function(event) {
				this.username = event.target.value
			},
			onPassword: function(event) {
				this.password = event.target.value
			},
			setting() {
				this.username = this.username ? this.username : this.userinfo.username;
				this.password = this.password ? this.password : this.userinfo.password;
				if(!this.username) {
					uni.showToast({
						icon: 'none',
						title: '请输入账号!'
					})
					return;
				}
				if(!this.password) {
					uni.showToast({
						icon: 'none',
						title: '请输入密码!'
					})
					return;
				}
				
				var api_url = this.$api_url;
				var username = this.username;
				var password = this.password;
				var sid = uni.getStorageSync('sid');
				uni.getStorage({
				    key: 'uid',
				    success: function (res) {
						var uid = res.data;
						if (!uid) {
							uni.showModal({
								title: '您没有登录！',
								content: '请先登录！',
								showCancel: false
							});
							return;
						}
						
						uni.request({
						    url: api_url + '/user/set_user', 
							method: 'POST',
							header: {'content-type': 'application/x-www-form-urlencoded'},
						    data: {username:username,password:password,sid:sid,uid:uid},
						    success: (res) => {
								console.log(res.data);
								if(res.data.code==200) {
									uni.showModal({
										title: '设置成功!',
										showCancel: false
									})
									
									uni.reLaunch({
									    url: '/pages/user/index'
									});
								} else {
									uni.showModal({
										title: res.data.msg,
										showCancel: false
									})
								}
						    },
							fail: (err) => {
								console.log('设置账号失败:', err);
							}
						});
				    }
				});
			}
		}
	}
</script>

<style>
	page,
	view {
		display: flex;
	}
</style>
