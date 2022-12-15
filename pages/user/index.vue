<template>
	<view class="new-users copy-data"
		:style="{height:pageHeight}">
		<view class="mid"
			style="flex:1;overflow: hidden;">
			<scroll-view scroll-y="true"
				style="height: 100%;">
				<view class="bg"></view>
				<view class="head pad30">
					<view class="user-card">
						<view class="user-info">
							<image class="avatar"
								:src='userInfo.avatar'
								v-if="userInfo.avatar && uid"
								@click="goEdit()"></image>
							<image v-else
								class="avatar"
								src="/static/images/f.png"
								mode=""
								@click="goEdit()"></image>
							<view class="info">
								<view class="name"
									v-if="!uid"
									@tap="openAuto"> 请点击登录 </view>
								<view class="name"
									v-if="uid">
									{{userInfo.nickname}}
									<view class="vip"
										v-if="userInfo.vip">
										<image :src="userInfo.vipIcon"
											alt="">
											<view style="margin-left: 10rpx;"
												class="vip-txt">{{userInfo.vipName || ''}}
											</view>
									</view>
								</view>
								<view class="num"
									v-if="userInfo.phone && uid"
									@click="goEdit()">
									<view class="num-txt">UID:{{uid}}</view>
									<view class="icon">
										<image src="/static/images/edit.png"
											mode=""></image>
										<view class="wenzi"> 修改资料 </view>
									</view>
								</view>
								<view class="phone"
									v-if="!userInfo.phone && isLogin"
									@tap="bindPhone">绑定手机号</view>
							</view>
						</view>
					</view>
				</view>
				<view class="contenBox">
					<!-- 会员菜单 -->
					<view class="user-menus"
						style="margin-top: 20rpx;">
						<view class="menu-title">我的服务</view>
						<view class="list-box">
							<block v-for="(item,index) in MyMenus"
								:key="index">
								<navigator class="item"
									:url="item.url"
									hover-class="none"
									v-if="!(item.url =='/pages/service/index' || (item.url =='/pages/users/user_spread_user/index' && !userInfo.isPromoter))">
									<image :src="item.pic"></image>
									<text>{{item.name}}</text>
								</navigator>
							</block>
							<!-- #ifndef MP -->
							<view class="item"
								@click="kefuClick">
								<image :src="servicePic"></image>
								<text>联系客服</text>
							</view>
							<!-- #endif -->
							<!-- #ifdef MP -->
							<button class="item"
								open-type='contact'
								hover-class='none'>
								<image :src="servicePic"></image>
								<text>联系客服</text>
							</button>
							<!-- #endif -->
						</view>
					</view>
					<view class="uni-p-b-98"></view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>
<script>
	let sysHeight = uni.getSystemInfoSync().statusBarHeight + 'px';
	import Cache from '@/utils/cache';
	import {
		BACK_URL
	} from '@/config/cache';
	import {
		getMenuList
	} from '@/api/user.js';
	import {
		orderData
	} from '@/api/order.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		getCity
	} from '@/api/api.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef H5
	import Auth from '@/libs/wechat';
	// #endif
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	const app = getApp();
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
		},
		computed: mapGetters(['isLogin', 'chatUrl', 'userInfo', 'uid']),
		data() {
			return {
				imgUrls: [],
				userMenu: [],
				autoplay: true,
				circular: true,
				interval: 3000,
				duration: 500,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				orderStatusNum: {},
				MyMenus: [{
					"wap_url": "/pages/users/user_vip/index",
					"name": "会员中心",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/dd497d333f4541d7ae41b2c579706370jhp2x39yrg.png",
					"id": 1238,
					"url": "/pages/users/user_vip/index"
				}, {
					"wap_url": "/pages/activity/bargain/index",
					"name": "砍价记录",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/5d6617d3440c4d67802fdf064b77b471qzfen9343d.png",
					"id": 1239,
					"url": "/pages/activity/bargain/index"
				}, {
					"wap_url": "/pages/users/user_spread_user/index",
					"name": "我的推广",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/197b96e3a4d24728b4069b0c288326352gcds3duga.png",
					"id": 1240,
					"url": "/pages/users/user_spread_user/index"
				}, {
					"wap_url": "/pages/users/user_money/index",
					"name": "我的余额",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/0624fe22d770438891ad666ebef9e21e1rhiepyec1.png",
					"id": 1241,
					"url": "/pages/users/user_money/index"
				}, {
					"wap_url": "/pages/users/user_address_list/index",
					"name": "地址信息",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/886e04146694474c966d346222fe2897ealwo6qycj.png",
					"id": 1242,
					"url": "/pages/users/user_address_list/index"
				}, {
					"wap_url": "/pages/users/user_goods_collection/index",
					"name": "我的收藏",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/3308f2d779874079b0e02ae206cf141bwb23ii2z52.png",
					"id": 1243,
					"url": "/pages/users/user_goods_collection/index"
				}, {
					"wap_url": "/pages/users/user_coupon/index",
					"name": "优惠券",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/11113e1876ac427bb3173086a3bd6c5bhyan1ea9pz.png",
					"id": 1244,
					"url": "/pages/users/user_coupon/index"
				}, {
					"wap_url": "https://yzf.qq.com/xv/web/static/chat/index.html?sign=37ef9b97db2656c32340cde61ce2b56a2176efe72ac7ed421c77607b5c816611ec4775a17c7605b33df1ffe1d22a4ce7464dd07b",
					"name": "联系客服",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/77ac54ce5f514ebbb89854f927f2a891dzefr2qxmh.png",
					"id": 1245,
					"url": "/pages/service/index"
				}],
				servicePic: '/static/images/customer.png',
				sysHeight: sysHeight,
				// #ifdef MP
				pageHeight: '100%',
				// #endif
				// #ifdef H5
				pageHeight: app.globalData.windowHeight,
				// #endif
				// #ifdef H5
				isWeixin: Auth.isWeixin()
				//#endif
			}
		},
		onLoad() {
			let that = this;
			console.log("that.userInfo:", that.userInfo);
			// #ifdef H5
			that.$set(that, 'pageHeight', app.globalData.windowHeight);
			// #endif
			if (!this.$Cache.has('cityList')) this.getCityList();
			if (that.isLogin == false) {
				// #ifdef H5
				toLogin()
				// #endif
			}
		},
		onShow: function() {
			let that = this;
			// #ifdef H5
			uni.getSystemInfo({
				success: function(res) {
					that.pageHeight = res.windowHeight + 'px'
				}
			});
			// #endif
			if (that.isLogin) {
				console.log("that.isLogin:", that.isLogin);
				this.$store.dispatch('USERINFO');
			} else {
				toLogin();
			}
		},
		methods: {
			navito(e) {
				window.location.href = 'https://' + e;
			},
			kefuClick() {
				location.href = this.chatUrl;
			},
			// 打开授权
			openAuto() {
				Cache.set(BACK_URL, '')
				toLogin();
			},
			// 授权回调
			onLoadFun() {
				this.getMyMenus();
			},
			Setting: function() {
				uni.openSetting({
					success: function(res) {
						console.log(res.authSetting)
					}
				});
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			// 绑定手机
			bindPhone() {
				uni.navigateTo({
					url: '/pages/users/app_login/index'
				})
			},
			// 编辑页面
			goEdit() {
				if (this.isLogin == false) {
					toLogin();
				} else {
					uni.navigateTo({
						url: '/pages/users/user_info/index'
					})
				}
			},
			// 获取地址数据
			getCityList: function() {
				let that = this;
				getCity().then(res => {
					console.log("获取地址数据:", res);
					let oneDay = 24 * 3600 * 1000;
					this.$Cache.setItem({
						name: 'cityList',
						value: res.data,
						expires: oneDay * 7
					}); //设置七天过期时间
				})
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
