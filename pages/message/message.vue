<template>
	<view class="">
		<view class="paddding_left_right_40">
			<z-message :list='list'></z-message>
		</view>
		<u-loadmore :status="status"
			:loading-text="loadingText"
			:loadmore-text="loadmoreText"
			:nomore-text="nomoreText" />
	</view>
</template>
<script>
	// import {
	// 	getMessageListOld
	// } from '@/api/api/index.js'
	export default {
		data() {
			return {
				res: {
					"code": 1,
					"msg": "消息列表",
					"time": "1671184932",
					"data": [{
						"id": 90103,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【Ⅰ级门票 - 通用】",
						"title": "拼桌成功",
						"createtime": "2022-12-08 14:31",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 87852,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【Ⅱ级门票 - 通用】",
						"title": "拼桌成功",
						"createtime": "2022-09-08 17:15",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 87694,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【元晶】",
						"title": "拼桌成功",
						"createtime": "2022-09-06 14:46",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 81400,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【戌狗】",
						"title": "拼桌成功",
						"createtime": "2022-08-11 17:18",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 81398,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【戌狗】",
						"title": "拼桌成功",
						"createtime": "2022-08-11 16:57",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 81089,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【苏州盲盒Ⅰ】",
						"title": "拼桌成功",
						"createtime": "2022-08-10 15:14",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 80610,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【九龙盲盒】",
						"title": "拼桌成功",
						"createtime": "2022-08-09 11:18",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 72914,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【辛弃疾和郁孤台】已经被购买了",
						"title": "拼桌成功",
						"createtime": "2022-08-03 18:42",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 72481,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了【城门变迁】已经被购买了",
						"title": "拼桌成功",
						"createtime": "2022-08-03 18:13",
						"user_id": 198007,
						"read_status": 1
					}, {
						"id": 72434,
						"content": "你恭喜你成功拼桌剧本杀活动恭喜你成功拼桌剧本杀活动买了 ",
						"title": "拼桌成功",
						"createtime": "2022-08-03 18:11",
						"user_id": 198007,
						"read_status": 1
					}]
				},
				list: [],
				loading: false,
				status: 'loading',
				loadingText: '努力加载中',
				loadmoreText: '轻轻上拉',
				nomoreText: '到底了',
			}
		},
		onLoad() {
			this.getList()
		},
		methods: {
			getList() {
				this.list = []
				this.status = 'loading'
				console.log(this.res)
				this.status = 'nomore'
				this.list = this.res.data;
				// getMessageListOld().then(res => {
				// 	// console.log(res)
				// 	this.status = 'nomore'
				// 	this.list = res.data;
				// 	this.loading = false
				// }).catch(err => {})
			}
		}
	}
</script>
<style lang="scss"
	scoped>
	page {
		font-family: PingFang SC;
	}

	.refresh {
		animation: loading 2s linear 0s infinite;
	}

	@keyframes loading {
		from {
			transform: rotate(0deg);
		}

		to {
			transform: rotate(360deg);
		}
	}

	.mess_tips_box {
		width: 750rpx;
		background: #121425;
		opacity: 1;
		padding: 15rpx 40rpx;
	}

	.go_btn {
		width: 128rpx;
		height: 52rpx;
		background: #3377FF;
		opacity: 1;
		border-radius: 26rpx;
		text-align: center;
		line-height: 52rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
		opacity: 1;
		font-size: 26rpx;
	}
</style>
