<template>
	<view class="about">
		<view class="content">
			<view class="qrcode">
				<!-- #ifdef APP-PLUS -->
				<image src="/static/app_qr_code.jpg" @longtap="save"></image>
				<!-- #endif -->
				<!-- #ifdef H5 -->
				<image src="/static/app_qr_code.jpg"></image>
				<!-- #endif -->
				<!-- #ifdef MP-WEIXIN -->
				<image src="/static/app_qr_code.jpg"></image>
				<!-- #endif -->
				<text class="tip">扫码体验看视频</text>
			</view>
			<!-- <view class="desc">
				知新闻，赏图片，阅书刊，看笑话，微航资讯与你一起让生活多份乐趣！
			</view>
			<view class="source">
				<view class="title">本产品内容：</view>
				<view class="source-list">
					<view class="source-cell">
						<text space="nbsp">1. </text>
						<text>最新新闻头条，各类资讯实时更新。</text>
					</view>
					<view class="source-cell">
						<text space="nbsp">2. </text>
						<text>精选图片专题，一起发现多彩的世界。</text>
					</view>
					<view class="source-cell">
						<!-- <text space="nbsp">2. </text><text @click="openLink" class="link">{{sourceLink}}</text> -->
						<!-- <text space="nbsp">3. </text>
						<text>书刊，网络幽默、搞笑、内涵段子，不间断。</text>
					</view>
				</view>
			</view> -->
			<!-- #ifdef H5 -->
			<button type="default" class="share" hover-class="hover" @click="download()">下载</button>
			<!-- #endif -->
			<!-- #ifdef MP-WEIXIN -->
			<button type="default" open-type="share" class="share" hover-class="hover">分享</button>
			<!-- <button type="default" open-type="contact" class="share" hover-class="hover">客服</button> -->
			<button type="default" class="share" hover-class="hover" @click="removeStorageSync()">清除缓存</button>
			<!-- #endif -->
			<!-- #ifdef APP-PLUS -->
			<view class="share-view" @click="goShare()">分享</view>
			<view class="share-view" @click="removeStorageSync()">清除登录缓存</view>
			<button type="default" :class="version_update?'update':'update_view'" hover-class="hover" @click="openLink()">版本更新</button>
			<!-- #endif -->
		</view>
		<!-- #ifdef APP-PLUS -->
		<view class="version">
			当前版本：{{version}}
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				providerList: [],
				version: '',
				news_version: '',
				version_update: false,
				//sourceLink: 'https://github.com/dcloudio/uni-template-picture'
			}
		},
		
		onLoad() {
			// #ifdef APP-PLUS
			this.version = plus.runtime.version;
			
			uni.request({
				url: this.$api_url + '/version/index',
				method: 'POST',
				header: {'content-type': 'application/x-www-form-urlencoded'},
				data: '',
				success: (result) => {
					if (result.statusCode !== 200) {
						return;
					}
					this.news_version = result.data.data;
					
					console.log(this.version);
					console.log(this.news_version);
					if(this.version!=this.news_version) {
						this.version_update = true;
					}
				},
				fail: (err) => {
					console.log('获取接口返回错误:', err);
				},
			});
			
			// 获取分享通道
			uni.getProvider({
				service: 'share',
				success: (e) => {
					let data = [];
					for (let i = 0; i < e.provider.length; i++) {
						switch (e.provider[i]) {
							case 'weixin':
								data.push({
									name: '分享到微信好友',
									id: 'weixin'
								})
								data.push({
									name: '分享到微信朋友圈',
									id: 'weixin',
									type: 'WXSenceTimeline'
								})
								break;
							case 'qq':
								data.push({
									name: '分享到QQ',
									id: 'qq'
								})
								break;
							default:
								break;
						}
					}
					this.providerList = data;
				},
				fail: (e) => {
					console.log('获取登录通道失败' + JSON.stringify(e));
				}
			});
			// #endif
		},
		
		onShareAppMessage(res) {
			return {
				title: '文字一键转成语音',
				path: '/pages/about/index'
			}
		},
		onNavigationBarButtonTap(e) {
			if (e.index === 0) {
				// #ifdef APP-PLUS
				if (this.providerList.length === 0) {
					uni.showModal({
						title: '当前环境无分享渠道!',
						showCancel: false
					})
					return;
				}
				let itemList = this.providerList.map(function(value) {
					return value.name
				})
				uni.showActionSheet({
					itemList: itemList,
					success: (res) => {
						uni.share({
							provider: this.providerList[res.tapIndex].id,
							scene: this.providerList[res.tapIndex].type && this.providerList[res.tapIndex].type === 'WXSenceTimeline' ?
								'WXSenceTimeline' : 'WXSceneSession',
							type: 0,
							title: '知新闻，赏图片，阅书刊，看笑话，微航资讯与你一起让生活多份乐趣！',
							summary: '最新新闻头条，各类资讯实时更新。精选图片专题，一起发现多彩的世界。网络幽默、搞笑、内涵段子，不间断。',
							//imageUrl: this.data[this.index],
							imageUrl: 'http://www.navculture.com/images/navculture_logo.png',
							href: 'http://www.navculture.com',
							success: (res) => {
								console.log('success:' + JSON.stringify(res));
							},
							fail: (e) => {
								uni.showModal({
									content: e.errMsg,
									showCancel: false
								})
							}
						});
						/* uni.share({
							provider: this.providerList[res.tapIndex].id,
							scene: this.providerList[res.tapIndex].type && this.providerList[res.tapIndex].type === 'WXSenceTimeline' ?
								'WXSenceTimeline' : "WXSceneSession",
							type: 0,
							title: '欢迎体验uni-app',
							summary: 'uni-app 是一个使用 Vue.js 开发跨平台应用的前端框架',
							imageUrl: 'https://img-cdn-qiniu.dcloud.net.cn/uploads/nav_menu/8.jpg',
							href: "https://m3w.cn/uniapp",
							success: (res) => {
								console.log("success:" + JSON.stringify(res));
							},
							fail: (e) => {
								uni.showModal({
									content: e.errMsg,
									showCancel: false
								})
							}
						}); */
					}
				});
				// #endif
			}
		},
		
		methods: {
			// #ifdef APP-PLUS
			save() {
				uni.showActionSheet({
					itemList: ['保存图片到相册'],
					success: () => {
						plus.gallery.save('/static/app_qr_code.jpg', function() {
							uni.showToast({
								title: '保存成功',
								icon: 'none'
							})
						}, function() {
							uni.showToast({
								title: '保存失败，请重试！',
								icon: 'none'
							})
						});
					}
				})
			},
			goShare() {
				if (this.providerList.length === 0) {
					uni.showModal({
						title: '当前环境无分享渠道!',
						showCancel: false
					})
					return;
				}
				let itemList = this.providerList.map(function(value) {
					return value.name
				})
				uni.showActionSheet({
					itemList: itemList,
					success: (res) => {
						uni.share({
							provider: this.providerList[res.tapIndex].id,
							scene: this.providerList[res.tapIndex].type && this.providerList[res.tapIndex].type === 'WXSenceTimeline' ?
								'WXSenceTimeline' : "WXSceneSession",
							type: 0,
							title: '知新闻，赏图片，阅书刊，看笑话，微航资讯与你一起让生活多份乐趣！',
							summary: '最新新闻头条，各类资讯实时更新。精选图片专题，一起发现多彩的世界。网络幽默、搞笑、内涵段子，不间断。',
							imageUrl: 'http://www.navculture.com/images/navculture_logo.png',
							href: "http://www.navculture.com",
							success: (res) => {
								console.log("success:" + JSON.stringify(res));
							},
							fail: (e) => {
								uni.showModal({
									content: e.errMsg,
									showCancel: false
								})
							}
						});
					}
				})
			},
			// #endif
			removeStorageSync() {
				uni.removeStorageSync('sid');
				uni.showToast({
					title: '清除完成',
					icon: 'none'
				})
			},
			download() {
				location.href = 'http://www.navculture.com/navculture.apk';
			},
			openLink() {
				var link = 'http://www.navculture.com/navculture.apk';
				if (plus) {
					plus.runtime.openURL(link);
				} else {
					window.open(link);
				}
			}
		}
	}
</script>

<style>
	page,
	view {
		display: flex;
	}

	page {
		min-height: 100%;
		background-color: #FFFFFF;
	}

	image {
		width: 360upx;
		height: 360upx;
	}

	.about {
		flex-direction: column;
		flex: 1;
	}

	.content {
		flex: 1;
		padding: 30upx;
		flex-direction: column;
		justify-content: center;
	}

	.qrcode {
		display: flex;
		align-items: center;
		flex-direction: column;
	}

	.qrcode .tip {
		margin-top: 20upx;
	}

	.desc {
		margin-top: 30upx;
		display: block;
	}

	.code {
		color: #e96900;
		background-color: #f8f8f8;
	}

	button {
		width: 100%;
		margin-top: 40upx;
	}
	
	button.share {
		background-color: #228FCD;
		color: #fff;
	}
	button.share.hover {
		opacity: 0.6;
	}
	
	button.update_view {
		background-color: #228FCD;
		color: #fff;
		display: none;
	}
	button.update {
		background-color: #228FCD;
		color: #fff;
	}
	button.update.hover {
		opacity: 0.6;
	}

	.version {
		height: 80upx;
		line-height: 80upx;
		justify-content: center;
		color: #ccc;
	}

	.source {
		margin-top: 30upx;
		flex-direction: column;
	}

	.source-list {
		flex-direction: column;
	}

	.link {
		color: #FF3300;
	}
	.share-view{
		justify-content: center;
		align-items: center;
		padding: 14upx 0;
		background-color: #228FCD;
		color: #FFF;
		border-radius: 5px;
		margin: 20upx 20upx 20upx;
		text-align: center;
		font-size: 38rpx;
		font-family: texticons;
		width: 95%;
		height: 30px;
		line-height: 30px;
	}
</style>
