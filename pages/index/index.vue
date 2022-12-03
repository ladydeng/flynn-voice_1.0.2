<template>
    <view>
        <!-- <view class="uni-title uni-common-pl"></view>
        <view class="uni-textarea">
            <textarea @blur="bindTextAreaBlur" auto-height />
        </view> -->
		
			<view class="uni-title uni-common-pl"></view>
            <view class="uni-textarea">
                <textarea @blur="bindTextAreaBlur" placeholder-style="color:#ccc" placeholder="请输入要转化的内容"/>
            </view>
			
			<view :class="src?'page-section page-section-gap':'page-section page-section-gap voice'" style="text-align: center;margin-top: 10px;">
				<audio style="text-align: left" :src="src" controls></audio>
			</view>
			
			<view style="margin-bottom: 5px;"></view>
			
			<!-- <ad unit-id="adunit-6679f45ff8a192a1"></ad> -->
			<ad unit-id="adunit-e8ae328e31e35247" ad-type="video" ad-theme="white"></ad>

			<view class="uni-title uni-common-pl"></view>
			<view class="uni-list">
				<view class="uni-list-cell">
					<view class="uni-list-cell-left">
						声音
					</view>
					<view class="uni-list-cell-db" style="text-align: center;">
						<picker @change="bindPickerChange" :value="per" :range="array">
							<view class="uni-input">{{array[per]}}</view>
						</picker>
					</view>
				</view>
			</view>

			<view class="uni-form-item uni-column">
				<view class="uni-title uni-common-pl">语速</view>
				<slider :value="spd" name="slider" @change="sliderChange" show-value></slider>
			</view>

			<button class="parsing" @click="change">转化</button>
			<button class="parsing" @click="download">下载</button>
			<button class="parsing" open-type="share">邀请好友</button>
			<button class="parsing" @click="get_image">聊天表情包</button>
			<button class="parsing" @click="removeWatermark">视频去水印</button>
			<button class="parsing" @click="go_video">在线看视频</button>
        
		<view class="faq">
		  <view class="faq-header">
		    <text class="faq-header-title">使用说明</text>
		  </view>
		  <view class="faq-content">
		    <view class="faq-content-list">
		      <text>1、每次把文字转换成语音消耗5积分，每天签到获得10积分。</text>
		    </view>
			<view class="faq-content-list">
			  <text>2、连续签到7天额外获得1000积分；连续签到15天额外获得2000积分；连续签到30天额外获得5000积分。</text>
			</view>
			<view class="faq-content-list" style="height:96px;">
			  <text>3、转发小程序给微信好友或微信群，好友点击使用(登录授权)后即算邀请成功；成功邀请1位好友获得1000积分，邀请好友及获取积分无上限。</text>
			</view>
		  </view>
		</view>
		
		<ad unit-id="adunit-6679f45ff8a192a1"></ad>
		<!-- <ad unit-id="adunit-e8ae328e31e35247" ad-type="video" ad-theme="white"></ad> -->
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				src: '',
				array: ['普通男声', '经典男声', '精致男声', '优雅女声', '文静女声'],
				tex: '',
				per: 4,
				spd: 10,
				params: {},
				inviter_uid: '',
				uid: ''
			}
		},
		onShareAppMessage(res) {
			if (res.from === 'menu') { // 来自右上角分享按钮
				console.log(res.target)
			}
			if (res.from === 'button') { // 来自页面内分享按钮
				console.log(res.target)
			}
			this.params.inviter_uid = this.uid;
			return {
				title: '文字一键转成语音',
				path: '/pages/index/index?query=' + encodeURIComponent(JSON.stringify(this.params))
			}
		},
		onLoad(e) {
			if(e.query!=undefined) {
				try {
					this.params = JSON.parse(decodeURIComponent(e.query));
					this.inviter_uid = this.params.inviter_uid;
				} catch (error) {
					this.params = JSON.parse(e.query);
					this.inviter_uid = this.params.inviter_uid;
				}
			}
			
			if(this.inviter_uid) {
				uni.setStorageSync('inviter_uid', this.inviter_uid);
			}
			
			this.uid = uni.getStorageSync('uid');
			console.log(this.uid);
			
		},
		methods: {
			bindTextAreaBlur: function (e) {
				this.tex = e.target.value;
				console.log(e.detail.value)
			},
			bindPickerChange: function(e) {
				this.per = e.target.value;
				console.log('picker发送选择改变，携带值为', e.target.value);
			},
			sliderChange(e) {
				this.spd = e.detail.value;
				console.log('value 发生变化：' + e.detail.value);
			},
			change() {
				if (!this.tex) {
					uni.showModal({
						title: '没有可转化的内容！',
						content: '请先输入要转化的内容！',
						showCancel: false
					});
					return;
				}
				
				this.src = 'https://tts.baidu.com/text2audio.mp3?tex='+this.tex+'&cuid=baike&amp&lan=ZH&amp&ctp=1&amp&pdt=301&amp&vol=100&amp&rate=32&amp&per='+this.per+'&spd='+this.spd+'&pit='+this.pit;
				
			},
			download() {
				if (!this.src) {
					uni.showToast({
						icon: 'none',
						title: '没有可下载的音频，内容转化成音频后才能下载!'
					})
					return;
				}
				
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				
				uni.request({
					url: this.$api_url + '/voice/download',
					method: 'POST',
					header: {'content-type': 'application/x-www-form-urlencoded'},
					data: {voice_url:this.src,sid:sid},
					success: (result) => {
						if (result.statusCode !== 200) {
							return;
						}
						
						if(result.data.code==200) {
							this.voice_name_url = result.data.data;
							
							const downloadTask = uni.downloadFile({
								url: this.voice_name_url,
								success: (e) => {
									uni.saveVideoToPhotosAlbum({
										filePath: e.tempFilePath,
										success: () => {
											uni.showToast({
												icon: 'none',
												title: '已保存到手机相册'
											})
										},
										fail: () => {
											uni.showToast({
												icon: 'none',
												title: '保存到手机相册失败' + e.tempFilePath
											})
										}
									});
								},
								fail: (e) => {
									uni.showModal({
										content: '下载失败，' + e.errMsg,
										showCancel: false
									})
								}
							});
							downloadTask.onProgressUpdate((res) => {
							    //console.log('下载进度' + res.progress);
							    //console.log('已经下载的数据长度' + res.totalBytesWritten);
							    //console.log('预期需要下载的数据总长度' + res.totalBytesExpectedToWrite);
								
								uni.showLoading({
								    title: '下载进度' + res.progress + '%'
								});
								
							    // 测试条件，取消下载任务。
							    //if (res.progress > 50) {
							        //downloadTask.abort();
							    //}
							})
							
						}else {
							uni.showToast({
								icon: 'none',
								title: result.data.msg,
							})
						}
						
						console.log(result.data);
					},
					fail: (err) => {
						console.log('获取接口返回错误:', err);
					},
				});
				
				this.src = '';
				console.log('下载');
			},
			removeWatermark() {
				uni.navigateToMiniProgram({
				    appId: 'wx0b42022e58e1222c',
				    path: 'pages/index/index',
				    success(res) {
						// 打开成功
				    }
				})
			},
			get_image() {
				uni.navigateToMiniProgram({
				    appId: 'wxf31807c4ce22d741',
				    path: 'pages/home/home',
				    success(res) {
						// 打开成功
				    }
				})
			},
			go_video() {
				uni.navigateToMiniProgram({
				    appId: 'wx2f5ef9271905d01f',
				    path: 'pages/index/index',
				    success(res) {
						// 打开成功
				    }
				})
			}
		}
	}
</script>

<style>
	page {
		background-color: #F4F5F6;
	}
	
	.voice {
		display: none;
	}
	
	.uni-title {
		font-size:40upx;
		font-weight:500;
		padding:20upx 0;
		line-height:1.5;
	}
	
	.uni-common-pl{
		padding-left:30upx;
	}
	
	.uni-list {
		background-color: #FFFFFF;
		position: relative;
		width: 100%;
		height:80upx;
		display: flex;
		flex-direction: column;
		line-height: 80upx;
	}
	.uni-list:after {
		position: absolute;
		z-index: 10;
		right: 0;
		bottom: 0;
		left: 0;
		height: 1px;
		content: '';
		-webkit-transform: scaleY(.5);
		transform: scaleY(.5);
		background-color: #c8c7cc;
	}
	.uni-list::before {
		position: absolute;
		z-index: 10;
		right: 0;
		top: 0;
		left: 0;
		height: 1px;
		content: '';
		-webkit-transform: scaleY(.5);
		transform: scaleY(.5);
		background-color: #c8c7cc;
	}
	.uni-list-cell {
		position: relative;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}
	.uni-list-cell-hover {
		background-color: #eee;
	}
	.uni-list-cell-pd {
		padding: 22upx 30upx;
	}
	.uni-list-cell-left {
		font-size:40upx;
		padding: 0 30upx;
	}
	.uni-list-cell-db,
	.uni-list-cell-right {
		flex: 1;
	}
	.uni-list-cell::after {
		position: absolute;
	    z-index: 3;
		right: 0;
		bottom: 0;
		left: 30upx;
		height: 1px;
		content: '';
		-webkit-transform: scaleY(.5);
		transform: scaleY(.5);
		background-color: #c8c7cc;
	}
	.uni-list .uni-list-cell:last-child::after {
		height: 0upx;
	}
	.uni-list-cell-last.uni-list-cell::after {
		height: 0upx;
	}
	.uni-list-cell-divider {
		position: relative;
		display: flex;
		color: #999;
		background-color: #f7f7f7;
		padding:15upx 20upx;
	}
	.uni-list-cell-divider::before {
		position: absolute;
		right: 0;
		top: 0;
		left: 0;
		height: 1px;
		content: '';
		-webkit-transform: scaleY(.5);
		transform: scaleY(.5);
		background-color: #c8c7cc;
	}
	.uni-list-cell-divider::after {
		position: absolute;
		right: 0;
		bottom: 0;
		left: 0upx;
		height: 1px;
		content: '';
		-webkit-transform: scaleY(.5);
		transform: scaleY(.5);
		background-color: #c8c7cc;
	}
	.uni-list-cell-navigate {
		font-size:30upx;
		padding: 22upx 30upx;
		line-height: 48upx;
		position: relative;
		display: flex;
		box-sizing: border-box;
		width: 100%;
		flex: 1;
		justify-content: space-between;
		align-items: center;
	}
	.uni-list-cell-navigate {
		padding-right: 36upx;
	}
	.uni-navigate-badge {
		padding-right: 50upx;
	}
	.uni-list-cell-navigate.uni-navigate-right:after {
		font-family: uniicons;
		content: '\e583';
		position: absolute;
		right: 24upx;
		top: 50%;
		color: #bbb;
		-webkit-transform: translateY(-50%);
		transform: translateY(-50%);
	}
	.uni-list-cell-navigate.uni-navigate-bottom:after {
		font-family: uniicons;
		content: '\e581';
		position: absolute;
		right: 24upx;
		top: 50%;
		color: #bbb;
		-webkit-transform: translateY(-50%);
		transform: translateY(-50%);
	}
	.uni-list-cell-navigate.uni-navigate-bottom.uni-active::after {
		font-family: uniicons;
		content: '\e580';
		position: absolute;
		right: 24upx;
		top: 50%;
		color: #bbb;
		-webkit-transform: translateY(-50%);
		transform: translateY(-50%);
	}
	.uni-collapse.uni-list-cell {
		flex-direction: column;
	}
	.uni-list-cell-navigate.uni-active {
		background: #eee;
	}
	.uni-list.uni-collapse {
		box-sizing: border-box;
		height: 0;
		overflow: hidden;
	}
	.uni-collapse .uni-list-cell {
		padding-left: 20upx;
	}
	.uni-collapse .uni-list-cell::after {
		left: 52upx;
	}
	.uni-list.uni-active {
		height: auto;
	}
	
	.uni-textarea{
		width:100%;
		background:#FFF;
	}
	.uni-textarea textarea{
		width:96%;
		padding:18upx 2%;
		line-height:1.6;
		font-size:28upx;
		height:300upx;
	}
	
	.parsing {
	    overflow: hidden;
	    display: block;
	    width: 664rpx;
	    height: 88rpx;
	    line-height: 88rpx;
	    color: #fff;
	    background-color: #228FCD;
	    box-shadow: 0 14rpx 28rpx rgba(51,122,255,.3);
		margin-top: 15px;
	}
	
	.faq {
	    width: 664rpx;
	    margin: 106rpx auto 0;
	}
	
	.faq-header {
	    display: flex;
	    justify-content: space-between;
	    margin-bottom: 16rpx;
	}
	
	.faq-header-title {
	    font-size: 28rpx;
	    font-weight: 600;
	}
	
	.faq-header-more {
	    font-size: 24rpx;
	    color: #989898;
	}
	
	.faq-content {
	    width: 100%;
	    border-radius: 12rpx;
	    background: #f6f6f6;
	}
	
	.faq-content-list {
	    overflow: hidden;
	    height: 108rpx;
	    line-height: 50rpx;
	}
	
	.faq-content-list text {
	    font-size: 26rpx;
	    color: #666;
	    margin-left: 32rpx;
	}
	
	.faq-content-list image {
	    float: right;
	    width: 16rpx;
	    height: 22rpx;
	    margin-top: 43rpx;
	    margin-right: 38rpx;
	}
</style>
