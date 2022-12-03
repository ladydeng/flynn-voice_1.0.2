<template>
	<view>
		<model-calendar 
		:sendYear="toYear" :sendMonth="toMonth"
		:dataSource="signData" :totalNum="sumCount"
		 @dateChange="getData" @clickChange="clickRegister"
		 ></model-calendar>
		<view class='count'>
			<text>截至目前，已坚持打卡</text>
			<view class='daynumber'>
				<text class='number'>{{sumCount}}</text>
				<text class='day'>天</text>
			</view>

			<view>累积获得积分<text class="monthSum">{{integralData}}</text>分</view>
			<text>请再接再厉，继续努力!</text>
		</view>
	</view>
</template>

<script>
	import modelCalendar from '@/components/calendar.vue';

	export default {
		data() {
			return {
				toYear: parseInt(new Date().getFullYear()), //本日
				toMonth: parseInt(new Date().getMonth() + 1), //本月
				sumCount: 0,
				signData: [],
				integralData:0
			};
		},
		components: {
			modelCalendar
		},
		created() {
			//获取当前用户当前任务的签到状态  			
			this.getData(this.toYear+"-"+this.toMonth);
		},
		methods: {
			clickRegister(day) {
				//console.log("在模版页签到了", day);
				this.signData.push(day);
				this.sumCount++;
				this.integralData = Number(this.integralData) + 10;
				
				//this.$http.postHttp("Comment/UpdateRecord", day, (res) => {//可以通过后台接口添加当前用户当日打卡记录，
				// 		//console.log(res);
				// 		//if (res!= undefined) {
				// 			uni.showToast({
				// 				title: "打卡成功"+day,
				// 				icon: 'success',
				// 				duration: 2000
				// 			});
				// 		
				// 		//}
				//  })	
			},
			//当模板的时候可以直接引入，然后触发子组件事件到父界面去控制数据

			//获取当前用户该任务的签到数组
			getData(val) {
				let y=val.split('-')[0];
				let m=val.split('-')[1];
				//this.$http.postHttp("Comment/GetRecord", {//可以通过后台接口去获取你的打卡数据
				// 	Year: y,
				// 	Month: m,
				// }, (res) => {
				//console.log(res);
				
				var sid = uni.getStorageSync('sid');
				if (!sid) {
					uni.showModal({
						title: '您没有登录！',
						content: '请先登录！',
						showCancel: false
					});
					return;
				}
				
				var ym_time = y + '-' + m;
				var uid = uni.getStorageSync('uid');
				uni.request({
					url: this.$api_url + '/user/get_sign',
					method: 'POST',
					header: {'content-type': 'application/x-www-form-urlencoded'},
					data: {ym_time:ym_time,uid:uid},
					success: (result) => {
						if (result.statusCode !== 200) {
							return;
						}
						this.sumCount = result.data.count;
						this.signData = result.data.time;
						this.integralData = result.data.integral;
						
						console.log(result.data);
					},
					fail: (err) => {
						console.log('获取接口返回错误:', err);
					},
				});
				
				
				//this.sumCount = 3; //res.SumCount		
				/* if (y == 2019 && m == 10) {
					this.signData = ["2019-10-01", "2019-10-02", "2019-10-03", "2019-10-04", "2019-10-07", "2019-10-08", "2019-10-09",
						"2019-10-16","2019-10-27"
					];
				} else {
					this.signData = [];
				} */
				// })
			},
		}
	}
</script>

<style lang='scss'>
	page {
		display: flex;
		flex-direction: column;
		background-color: #FFFFFF;
	}
	
	.count .daynumber {
		display: flex;
		flex-direction: row;
		justify-content: center;
	}

	.count .daynumber .day {
		margin-top: 50rpx;
	}

	.count {
		margin: 20rpx;
		padding: 30rpx;
		display: flex;
		text-align: center;
		border-radius: 10px;
		flex-direction: column;
		justify-content: center;
		background-color: #fff;
		align-items: center;
	}

	.count .number {
		color: #fff;
		font-size: 60rpx;
		background-color: #94db98;
		width: 100rpx;
		height: 100rpx;
		border-radius: 50%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		margin: 20rpx;
	}

	.monthSum {
		color: red;
		font-size: 40rpx;
	}

	.count text {
		margin: 10rpx;
	}
</style>
