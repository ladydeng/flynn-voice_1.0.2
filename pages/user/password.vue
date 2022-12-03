<template>
	<form class='loginView' @submit="setting">
		<view class="input-view">
			<view class="label-view" style="width:90px;">
				<text class="label">原密码 </text>
			</view>
			<input class="input" type="password" placeholder="请输入原密码" name="old_password" />
		</view>
		<view class="input-view">
			<view class="label-view" style="width:90px;">
				<text class="label">新密码</text>
			</view>
			<input class="input" type="password" placeholder="请输入新密码" name="new_password" />
		</view>
		<view class="input-view">
			<view class="label-view" style="width:90px;">
				<text class="label">确认密码</text>
			</view>
			<input class="input" type="password" placeholder="请再次输入新密码" name="password" />
		</view>
		<view class="button-view">
			<button type="default" class="confirm" hover-class="hover" formType="submit">确认</button>
		</view>
	</form>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
		methods: {
			setting(e) {
				var old_password = e.detail.value.old_password;
				var new_password = e.detail.value.new_password;
				var password = e.detail.value.password;
				
				if(!old_password) {
					uni.showToast({
						icon: 'none',
						title: '请输入原密码!'
					})
					return;
				}
				if(!new_password) {
					uni.showToast({
						icon: 'none',
						title: '请输入新密码!'
					})
					return;
				}
				if(!password) {
					uni.showToast({
						icon: 'none',
						title: '请输入确认密码!'
					})
					return;
				}
				if(new_password!=password) {
					uni.showToast({
						icon: 'none',
						title: '密码输入不一致,请重新输入!'
					})
					return;
				}
				
				var api_url = this.$api_url;
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
						    url: api_url + '/user/edit_password', 
							method: 'POST',
							header: {'content-type': 'application/x-www-form-urlencoded'},
						    data: {old_password:old_password,new_password:new_password,password:password,sid:sid,uid:uid},
						    success: (res) => {
								console.log(res.data)
								if(res.data.code==200) {
									uni.showToast({
										icon: 'none',
										title: '修改成功!'
									})
									
									/* uni.reLaunch({
									    url: '/pages/user/index'
									}); */
								} else {
									uni.showToast({
										icon: 'none',
										title:  res.data.msg,
									})
									/* uni.showModal({
										title: res.data.msg,
										showCancel: false
									}) */
								}
						    },
							fail: (err) => {
								console.log('修改密码失败:', err);
							}
						});
				    }
				});
				//uni.navigateTo({
					//url: '/pages/register/register'
					
				//});
			}
		}
	}
</script>

<style>
	page,
	view {
		display: flex;
	}
	
	button.confirm {
		background-color: #228FCD;
		color: #fff;
	}
</style>
