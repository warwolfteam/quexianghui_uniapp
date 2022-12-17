<template>
	<view class="new-users copy-data">
		<view class="head pad30">
			<view class="user-card">
				<view class="user-info">
					<image class="avatar"
						:src='userInfo.avatar'
						v-if="userInfo.avatar && uid"></image>
					<view class="info">
						<view class="name">
							{{userInfo.nickname}}
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="my_body">
			<u-cell-group title="项目名称：剧本杀">
				<u-cell-item title="限制条件"
					value="仅限女士"
					:arrow="false"></u-cell-item>
				<u-cell-item title="类型"
					value="AA制"
					:arrow="false"></u-cell-item>
				<u-cell-item title="时间："
					:arrow="false"
					value="2022.12.12"></u-cell-item>
				<u-cell-item title="地点："
					:arrow="false"
					value="步步高7楼"></u-cell-item>
				<u-cell-item title="备注："
					:arrow="false"
					value="XXXXXXXXXXX"></u-cell-item>
			</u-cell-group>
		</view>
		<view class="ganxingqu">
			<u-button :custom-style="customStyle"
				@click="goPinZhuo()"
				size="medium">我感兴趣</u-button>
		</view>
		<view class="zhanwei">
		</view>
	</view>
</template>
<script>
	import {
		mapGetters
	} from "vuex";
	export default {
		data() {
			return {
				customStyle: {
					marginTop: '-20rpx', // 注意驼峰命名，并且值必须用引号包括，因为这是对象
					color: '#ff0053',
					backgroundColor: "rgb(250 227 227)",
				},
				info: {
					activity: null,
					activityH5: null,
					cartNum: null,
					ficti: null,
					flatPattern: null,
					id: null,
					image: null,
					otPrice: null,
					price: null,
					sales: null,
					stock: null,
					storeName: null,
					unitName: null,
				},
			};
		},
		computed: mapGetters(['isLogin', 'chatUrl', 'userInfo', 'uid']),
		onLoad(option) {
			var that = this;
			console.log("输出传入参数：", option);
			const item = JSON.parse(decodeURIComponent(option.item));
			console.log("格式化输出传入参数：", item);
			that.info = item;
		},
		methods: {
			goPinZhuo() {
				console.log("点击感兴趣");
			}
		}
	}
</script>
<style lang="scss"
	scoped>
	page,
	body {
		height: 100%;
	}

	.bg {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 420rpx;
		background-image: url('~@/static/images/user_bg.png');
		background-repeat: no-repeat;
		background-size: 100% 100%;
	}

	.contenBox {
		padding: 0 30rpx;
		position: relative;
		z-index: 999;
	}

	.support {
		width: 219rpx;
		height: 74rpx;
		margin: 54rpx auto;
		display: block;
	}

	.justify-between {
		justify-content: space-between;
	}

	.mar_top30 {
		margin-top: 15px;
	}

	.color_000 {
		color: #000;
	}

	.uni-scroll-view,
	.uni-scroll-view-content {
		width: 100%;
		height: 100%;
	}

	.flex {
		display: flex;
	}

	.fontS32 {
		font-size: 32rpx;
	}

	.collection {
		margin: 5%;

		.collection_title {
			.select_box {}
		}

		.collectionList {
			// width: 50rpx;
			margin-top: 16rpx;

			image {
				width: 236rpx;
				height: 236rpx;
				margin-right: 30rpx;
				border-radius: 20rpx;
			}
		}
	}

	.my_body {
		margin: 30rpx;
		padding: 15px;
		border-radius: 30rpx;
		text-align: center;
		background: #ffffff;
	}

	.ganxingqu {
		margin: 30rpx;
		padding: 15px;
		border-radius: 30rpx;
		text-align: center;
	}

	.zhanwei {
		height: 100rpx;
	}

	.new-users {
		display: flex;
		flex-direction: column;
		height: 100%;

		.sys-head {
			position: relative;
			width: 100%;
			background: linear-gradient(90deg, $bg-star1 0%, $bg-end1 100%);

			.sys-title {
				z-index: 10;
				position: relative;
				height: 43px;
				text-align: center;
				line-height: 43px;
				font-size: 36rpx;
				color: #FFFFFF;
			}
		}

		.head {
			background: linear-gradient(360deg, rgba(255, 121, 49, 0) 0%, rgba(248, 74, 29, 0.82) 39%, #E93323 100%);

			// padding: 0 30rpx;
			.user-card {
				position: relative;
				width: 100%;
				margin: 0 auto;
				padding: 35rpx 0 30rpx 0;

				.user-info {
					z-index: 20;
					position: relative;
					display: flex;

					.avatar {
						width: 120rpx;
						height: 120rpx;
						border-radius: 50%;
					}

					.info {
						flex: 1;
						display: flex;
						flex-direction: column;
						justify-content: space-between;
						margin-left: 20rpx;
						padding: 15rpx 0;

						.name {
							display: flex;
							align-items: center;
							color: #fff;
							font-size: 31rpx;

							.vip {
								display: flex;
								align-items: center;
								padding: 6rpx 20rpx;
								background: rgba(0, 0, 0, 0.2);
								border-radius: 18px;
								font-size: 20rpx;
								margin-left: 12rpx;

								image {
									width: 27rpx;
									height: 27rpx;
								}
							}
						}

						.num {
							float: right;
							display: flex;
							justify-content: space-between;
							align-items: center;
							font-size: 26rpx;
							color: rgba(255, 255, 255, 0.6);

							.num-txt {
								font-weight: 1000;
							}

							.icon {
								padding-top: 20rpx;
								display: flex;
								height: 70rpx;

								image {
									width: 22rpx;
									height: 23rpx;
									margin-left: 5rpx;
								}

								.wenzi {
									line-height: 20rpx;
								}
							}
						}
					}
				}

				.num-wrapper {
					z-index: 30;
					position: relative;
					display: flex;
					align-items: center;
					justify-content: space-between;
					margin-top: 30rpx;
					color: #fff;

					.num-item {
						width: 33.33%;
						text-align: center;

						.num {
							font-size: 42rpx;
							font-weight: bold;
						}

						.txt {
							margin-top: 10rpx;
							font-size: 26rpx;
							color: rgba(255, 255, 255, 0.6);
						}
					}
				}

				.sign {
					z-index: 200;
					position: absolute;
					right: -12rpx;
					top: 80rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					width: 120rpx;
					height: 60rpx;
					background: linear-gradient(90deg, rgba(255, 225, 87, 1) 0%, rgba(238, 193, 15, 1) 100%);
					border-radius: 29rpx 4rpx 4rpx 29rpx;
					color: #282828;
					font-size: 28rpx;
					font-weight: bold;
				}
			}

			.order-wrapper {
				background-color: #fff;
				border-radius: 14rpx;
				padding: 30rpx 16rpx;
				position: relative;
				z-index: 11;

				.order-hd {
					justify-content: space-between;
					font-size: 30rpx;
					color: #282828;
					margin-bottom: 40rpx;
					padding: 0 16rpx;

					.left {
						color: #282828;
						font-size: 30rpx;
						font-weight: 600;
					}

					.right {
						align-items: center;
						color: #666666;
						font-size: 26rpx;

						.icon-xiangyou {
							margin-left: 5rpx;
							font-size: 24rpx;
						}
					}
				}

				.order-bd {
					display: flex;
					justify-content: space-between;
					padding: 0;

					.order-item {
						display: flex;
						flex-direction: column;
						justify-content: center;
						align-items: center;

						.pic {
							position: relative;
							text-align: center;

							image {
								width: 48rpx;
								height: 48rpx;
							}
						}

						.txt {
							margin-top: 15rpx;
							font-size: 26rpx;
							color: #454545;
						}
					}
				}
			}
		}

		.slider-wrapper {
			margin: 20rpx 0;
			height: 138rpx;

			swiper,
			swiper-item {
				height: 100%;
			}

			image {
				width: 100%;
				height: 100%;
			}
		}

		.user-menus {
			background-color: #fff;
			border-radius: 14rpx;

			.menu-title {
				padding: 30rpx 30rpx 40rpx;
				font-size: 30rpx;
				color: #282828;
				font-weight: 600;
			}

			.list-box {
				display: flex;
				flex-wrap: wrap;
				padding: 0;
			}

			.item {
				position: relative;
				display: flex;
				align-items: center;
				justify-content: space-between;
				flex-direction: column;
				width: 25%;
				margin-bottom: 47rpx;
				font-size: 26rpx;
				color: #333333;

				image {
					width: 52rpx;
					height: 52rpx;
					margin-bottom: 18rpx;
				}

				&:last-child::before {
					display: none;
				}
			}

			button {
				font-size: 28rpx;
			}
		}

		.phone {
			color: #fff;
		}

		.order-status-num {
			min-width: 12rpx;
			background-color: #fff;
			color: #ee5a52;
			border-radius: 15px;
			position: absolute;
			right: -14rpx;
			top: -15rpx;
			font-size: 20rpx;
			padding: 0 8rpx;
			border: 1px solid #ee5a52;
		}
	}
</style>
