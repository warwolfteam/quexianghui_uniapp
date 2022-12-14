<template>
	<view>
		<view class="page-index"
			:class="{'bgf':navIndex >0}">
			<view class='search acea-row row-between-wrapper'>
				<view class='input acea-row row-between-wrapper'>
					<text class='iconfont icon-sousuo2'></text>
					<input type='text'
						:value='searchValue'
						:focus="focus"
						placeholder='点击搜索商品'
						placeholder-class='placeholder'
						@input="setValue"></input>
				</view>
				<view class='bnt'
					@tap='searchBut'>搜索</view>
			</view>
			<!-- 首页展示 -->
			<view class="page_content"
				:style="'margin-top:'+(marTop)+'px;'"
				v-if="navIndex == 0">
				<!-- 首页推荐 -->
				<!-- :class="iSshowH?'on':''" -->
				<view class="sticky-box"
					:style="'top:'+(marTop)+'px;'">
					<scroll-view class="scroll-view_H"
						style="width: 100%;"
						scroll-x="true"
						scroll-with-animation
						:scroll-left="tabsScrollLeft"
						@scroll="scroll">
						<view class="tab nav-bd"
							id="tab_list">
							<view id="tab_item"
								:class="{ 'active': listActive == index}"
								class="item"
								v-for="(item, index) in explosiveMoney"
								:key="index"
								@click="ProductNavTab(item,index)">
								<view class="txt">
									<u-icon :name="item.icon"
										color="#000000"
										size="48"></u-icon>
								</view>
								<view class="label">{{item.info}}</view>
							</view>
						</view>
					</scroll-view>
				</view>
				<!-- 首发新品 -->
				<view class="index-product-wrapper"
					:class="iSshowH?'on':''">
					<view class="list-box animated"
						:class='tempArr.length > 0?"fadeIn on":""'>
						<view class="item"
							v-for="(item,index) in tempArr"
							:key="index"
							@click="goDetail(item)">
							<view class="pictrue">
								<image :src="item.image"
									mode=""></image>
							</view>
							<view class="text-info">
								<view class="title line1">{{item.storeName}}</view>
								<!-- <view class="old-price"><text>¥{{item.otPrice}}</text></view> -->
								<view class="price">
									<text>￥</text>{{item.price}}
									<!-- <view class="txt"
										v-if="item.checkCoupon">券</view> -->
								</view>
							</view>
						</view>
					</view>
					<view class='loadingicon acea-row row-center-wrapper'
						v-if="goodScroll">
						<text class='loading iconfont icon-jiazai'
							:hidden='loading==false'></text>
					</view>
					<view class="mores-txt flex"
						v-if="!goodScroll">
						<text>我是有底线的</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	import Auth from '@/libs/wechat';
	import Cache from '@/utils/cache';
	var statusBarHeight = uni.getSystemInfoSync().statusBarHeight + 'px';
	let app = getApp();
	import {
		getIndexData,
		getCoupons,
		setCouponReceive
	} from '@/api/api.js';
	// #ifdef MP-WEIXIN
	import {
		getTemlIds
	} from '@/api/api.js';;
	// #endif
	// #ifdef H5  
	import {
		follow
	} from '@/api/public.js';
	// #endif
	import {
		getShare
	} from '@/api/public.js';
	import goodList from '@/components/goodList';
	import promotionGood from '@/components/promotionGood';
	import couponWindow from '@/components/couponWindow';
	import ClipboardJS from "@/plugin/clipboard/clipboard.js";
	import {
		goShopDetail
	} from '@/libs/order.js'
	import {
		mapGetters
	} from "vuex";
	import tabNav from '@/components/tabNav.vue'
	import countDown from '@/components/countDown';
	import {
		getCategoryList,
		getProductslist,
		getProductHot,
		getGroomList
	} from '@/api/store.js';
	import recommend from '@/components/recommend';
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import {
		silenceBindingSpread
	} from '@/utils';
	// #ifndef MP
	import {
		kefuConfig
	} from "@/api/public";
	import {
		getWechatConfig
	} from "@/api/public";
	// #endif
	import Loading from '@/components/Loading/index.vue';
	const arrTemp = ["beforePay", "afterPay", "refundApply", "beforeRecharge", "createBargain", "pink"];
	export default {
		computed: mapGetters(['isLogin', 'uid']),
		components: {
			tabNav,
			goodList,
			promotionGood,
			couponWindow,
			countDown,
			recommend,
			// #ifdef MP
			authorize,
			// #endif
			Loading
		},
		data() {
			return {
				loaded: false,
				loading: false,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				statusBarHeight: statusBarHeight,
				navIndex: 0,
				navTop: [],
				followUrl: "",
				followHid: true,
				followCode: false,
				itemNew: [],
				bastInfo: '',
				fastInfo: '',
				fastList: [],
				firstInfo: '',
				salesInfo: '',
				indicatorDots: false,
				circular: true,
				autoplay: true,
				interval: 3000,
				duration: 500,
				window: false,
				iShidden: false,
				navH: "",
				newGoodsBananr: '',
				couponList: [],
				liveList: [],
				hotList: [{
					pic: '/static/images/hot_001.png'
				}, {
					pic: '/static/images/hot_002.png'
				}, {
					pic: '/static/images/hot_003.png'
				}],
				ProductNavindex: 0,
				marTop: -1,
				childID: 0,
				loadend: false,
				loadTitle: '加载更多',
				sortProduct: [],
				where: {
					cid: 0,
					page: 1,
					limit: 10,
				},
				is_switch: true,
				hotPage: 1,
				hotLimit: 10,
				hotScroll: false,
				explosiveMoney: [{
					"id": 1247,
					"info": "全部",
					"icon": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2022/12/14/2ffd33566ae94e69a15b0e8c1560083ayebdw75tlm.png",
					"type": "1"
				}, {
					"id": 1248,
					"info": "美食",
					"icon": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2022/12/14/0557711503544dbd80c856fced63c729aynx9ecqvo.png",
					"type": "2"
				}, {
					"id": 1249,
					"info": "电影",
					"icon": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2022/12/14/873f9a1e16564259a92a90435ab891c9jfvv33dncf.png",
					"type": "3"
				}, {
					"id": 1246,
					"info": "唱歌",
					"icon": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2022/12/14/2f873174bc4546f4b7cc5dacf2609ab9t6h7xom6eg.png",
					"type": "4"
				}],
				prodeuctTop: 0,
				searchH: 0,
				isFixed: false,
				goodType: 0, //精品推荐Type
				goodScroll: true, //精品推荐开关
				params: { //精品推荐分页
					page: 1,
					limit: 10,
				},
				tempArr: [], //精品推荐临时数组
				roll: [], // 新闻简报
				iSshowH: false,
				configApi: {}, //分享类容配置
				spikeList: [], // 秒杀
				point: '',
				privacyStatus: false, // 隐私政策是否同意过
				tabsScrollLeft: 0, // tabs当前偏移量
				scrollLeft: 0,
				lineColor: 'red',
				lineStyle: {}, // 下划线位置--动态甲酸
				listActive: 0, // 当前选中项
				duration: 0.2, // 下划线动画时长
				searchValue: ""
			}
		},
		watch: {
			ProductNavindex(newVal) { // 监听当前选中项
				this.setTabList()
			},
			listActive(newVal) { // 监听当前选中项
				this.setTabList()
			}
		},
		mounted() {
			this.setTabList()
		},
		onLoad() {
			var that = this;
			// 获取系统信息
			uni.getSystemInfo({
				success(res) {
					that.$store.commit("SYSTEM_PLATFORM", res.platform);
				}
			});
			uni.getLocation({
				type: 'gcj02',
				altitude: true,
				geocode: true,
				success: function(res) {
					try {
						uni.setStorageSync('user_latitude', res.latitude);
						uni.setStorageSync('user_longitude', res.longitude);
					} catch {}
				}
			});
			let self = this
			// #ifdef MP
			// 获取小程序头部高度
			this.navH = app.globalData.navHeight;
			let info = uni.createSelectorQuery().select(".mp-header");
			// info.boundingClientRect(function(data) {
			// 	console.log("boundingClientRect:",info);
			// 	self.marTop = data.height
			// 	self.poTop = Number(data.height) + 84
			// }).exec()
			// #endif
			// #ifndef MP
			this.navH = 0;
			self.marTop = 42;
			// #endif
			this.isLogin && silenceBindingSpread();
			// Promise.all([this.getAllCategory(), this.getIndexConfig()
			// 	// , this.setVisit()
			// ]);
			this.getIndexConfig();
			// #ifdef MP
			this.getTemlIds()
			// #endif
		},
		onShow() {
			let self = this
		},
		methods: {
			getCoupon: function(id, index) {
				let that = this;
				//领取优惠券
				setCouponReceive(id).then(function(res) {
					that.$set(that.couponList[index], 'isUse', true);
					that.$util.Tips({
						title: '领取成功'
					});
				}, function(res) {
					return that.$util.Tips({
						title: res
					});
				})
			},
			clickSort(index) {
				this.listActive = index
			},
			// scroll-view滑动事件
			scroll(e) {
				this.scrollLeft = e.detail.scrollLeft;
			},
			setTabList() {
				this.$nextTick(() => {
					//this.setLine()
					this.scrollIntoView()
				})
			},
			// 计算tabs位置
			scrollIntoView() { // item滚动
				let lineLeft = 0;
				this.getElementData('#tab_list', (data) => {
					let list = data[0]
					this.getElementData(`#tab_item`, (data) => {
						let el = data[this.listActive]
						lineLeft = el.width / 2 + (-list.left) + el.left - list.width / 2 - this.scrollLeft
						this.tabsScrollLeft = this.scrollLeft + lineLeft
					})
				})
			},
			//  计算下划线位置
			setLine() {
				let lineWidth = 0,
					lineLeft = 0
				this.getElementData(`#tab_item`, (data) => {
					let el = data[this.listActive]
					lineWidth = el.width / 2
					// lineLeft = el.width * (this.currentIndex + 0.5)  // 此种只能针对每个item长度一致的
					lineLeft = el.width / 2 + (-data[0].left) + el.left
					this.lineStyle = {
						width: `${lineWidth}px`,
						transform: `translateX(${lineLeft}px) translateX(-50%)`,
						transitionDuration: `${this.duration}s`
					};
				})
			},
			getElementData(el, callback) {
				uni.createSelectorQuery().in(this).selectAll(el).boundingClientRect().exec((data) => {
					callback(data[0]);
				});
			},
			xieyiApp() {
				uni.navigateTo({
					url: '/pages/users/web_page/index?webUel=https://admin.java.crmeb.net/useragreement/xieyi.html&title=协议内容'
				})
			},
			// #ifdef MP
			getTemlIds() {
				for (var i in arrTemp) {
					this.getTem(arrTemp[i]);
				}
			},
			getTem(data) {
				getTemlIds({
					type: data
				}).then(res => {
					if (res.data) {
						let arr = res.data.map((item) => {
							return item.tempId
						})
						wx.setStorageSync('tempID' + data, arr);
					}
				})
			},
			// #endif
			// 首页数据
			getIndexConfig: function() {
				let that = this;
				getIndexData().then(res => {
					that.$set(that, "roll", res.data.roll ? res.data.roll : []);
					// #ifdef H5
					that.$store.commit("SET_CHATURL", res.data.yzfUrl);
					Cache.set('chatUrl', res.data.yzfUrl);
					// #endif
					// that.$set(that, "explosiveMoney", res.data.explosiveMoney);
					that.goodType = res.data.explosiveMoney[0].type
					this.getGroomList();
					this.shareApi();
					this.getcouponList();
					// #ifdef H5
					// that.subscribe = res.data.subscribe;
					// #endif
				})
			},
			getcouponList() {
				let that = this;
				getCoupons({
					page: 1,
					limit: 6
				}).then(res => {
					that.$set(that, "couponList", res.data);
					// 小程序判断用户是否授权；
					// #ifdef MP
					uni.getSetting({
						success(res) {
							if (!res.authSetting['scope.userInfo']) {
								that.window = that.couponList.length ? true : false;
							} else {
								that.window = false;
								that.iShidden = true;
							}
						}
					});
					// #endif
					// #ifndef MP
					if (that.isLogin) {
						that.window = false;
					} else {
						that.window = res.data.length ? true : false;
					}
					// #endif
				}).catch(err => {
					return this.$util.Tips({
						title: err
					});
				});
			},
			shareApi: function() {
				getShare().then(res => {
					this.$set(this, 'configApi', res.data);
					// #ifdef H5
					this.setOpenShare(res.data);
					// #endif
				})
			},
			getChatUrL() {
				kefuConfig().then(res => {
					let data = res.data;
					this.$store.commit("SET_CHATURL", data.yzfUrl);
					Cache.set('chatUrl', data.yzfUrl);
				})
			},
			// 微信分享；
			setOpenShare: function(data) {
				let that = this;
				if (that.$wechat.isWeixin()) {
					let configAppMessage = {
						desc: data.synopsis,
						title: data.title,
						link: location.href,
						imgUrl: data.img
					};
					that.$wechat.wechatEvevt(["updateAppMessageShareData", "updateTimelineShareData"],
						configAppMessage);
				}
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			// 授权回调
			onLoadFun() {},
			// 首发新品切换
			ProductNavTab(item, index) {
				console.log("ProductNavTab-item:", item, );
				console.log("ProductNavTab-index:", index, );
				this.searchValue = ""
				this.params.searchValue = ""
				this.listActive = index
				this.goodType = item.type
				this.listActive = index
				this.ProductNavindex = index
				this.tempArr = []
				this.params.page = 1
				this.goodScroll = true
				let onloadH = true
				this.getGroomList(onloadH)
			},
			setValue: function(event) {
				this.$set(this, 'searchValue', event.detail.value);
				console.log("searchValue", event.detail.value);
			},
			searchBut: function() {
				let that = this;
				that.focus = false;
				console.log("that.searchValue.length:", that.searchValue.length);
				if (that.searchValue.length > 0) {
					that.listActive = 0
					that.goodType = 1
					that.tempArr = []
					that.params.searchValue = that.searchValue;
					that.params.page = 1;
					that.loadend = false;
					uni.showLoading({
						title: '正在搜索中'
					});
					console.log("that.params:", that.params);
					that.goodScroll = true
					let onloadH = true
					that.getGroomList(onloadH);
					console.log("onloadH:", onloadH);
					uni.hideLoading();
				} else {
					return this.$util.Tips({
						title: '请输入要搜索的商品',
						icon: 'none',
						duration: 1000,
						mask: true,
					});
				}
			},
			// 首发新品详情
			goDetail(item) {
				if (item.activityH5 && item.activityH5.type === "2" && !this.isLogin) {
					toLogin();
				} else {
					goShopDetail(item, this.uid).then(res => {
						uni.navigateTo({
							url: `/pages/goods_details/index?id=${item.id}`
						})
					})
				}
			},
			// 分类详情
			godDetail(item) {
				goShopDetail(item, this.uid).then(res => {
					uni.navigateTo({
						url: `/pages/goods_details/index?id=${item.id}`
					})
				})
			},
			// 精品推荐
			getGroomList(onloadH) {
				console.log("进入接口请求环节");
				this.loading = true
				let type = this.goodType;
				console.log("this.goodScroll", this.goodScroll);
				if (!this.goodScroll) return
				if (onloadH) {
					this.iSshowH = true
				}
				getGroomList(type, this.params).then(({
					data
				}) => {
					this.iSshowH = false
					this.loading = false
					this.goodScroll = data.list.length >= this.params.limit
					this.params.page++
					this.tempArr = this.tempArr.concat(data.list)
				})
			},
			/**
			 * 获取我的推荐
			 */
			get_host_product: function() {
				let that = this;
				that.loading = true;
				if (that.hotScroll) return
				getProductHot(that.hotPage, that.hotLimit, ).then(res => {
					that.hotPage++
					that.hotScroll = res.data.list.length < that.hotLimit
					that.hostProduct = that.hostProduct.concat(res.data.list)
				});
			},
		},
		mounted() {
			let self = this
			// #ifdef H5
			//self.getChatUrL();
			// 获取H5 搜索框高度
			let appSearchH = uni.createSelectorQuery().select(".serch-wrapper");
			appSearchH.boundingClientRect(function(data) {
				self.searchH = data.height
			}).exec()
			// #endif
		},
		/**
		 * 用户点击右上角分享
		 */
		// #ifdef MP
		onShareAppMessage: function() {
			return {
				title: this.configApi.title,
				imageUrl: this.configApi.img,
				desc: this.configApi.synopsis,
				path: '/pages/index/index'
			};
		},
		// #endif
		onReachBottom() {
			if (this.navIndex == 0) {
				// 首页加载更多
				if (this.params.page != 1) {
					this.getGroomList();
				}
			} else {
				// 分类栏目加载更多
				if (this.sortProduct.length > 0) {
					//this.get_product_list();
				} else {
					this.get_host_product();
				}
			}
		},
	}
</script>
<style>
	page {
		display: flex;
		flex-direction: column;
		height: 100%;
		/* #ifdef H5 */
		background-color: #fff;
		/* #endif */
	}
</style>
<style lang="scss">
	.notice {
		width: 100%;
		height: 70rpx;
		border-radius: 10rpx;
		background-color: #fff;
		margin-bottom: 25rpx;
		line-height: 70rpx;
		padding: 0 14rpx;

		.line {
			color: #CCCCCC;
		}

		.pic {
			width: 130rpx;
			height: 36rpx;

			image {
				width: 100%;
				height: 100%;
				display: block !important;
			}
		}

		.swipers {
			height: 100%;
			width: 444rpx;
			overflow: hidden;

			swiper {
				height: 100%;
				width: 100%;
				overflow: hidden;
				font-size: 22rpx;
				color: #333333;
			}
		}

		.iconfont {
			color: #999999;
			font-size: 20rpx;
		}
	}

	.couponIndex {
		width: auto;
		height: 238rpx;
		background-image: url('~@/static/images/yhjsy.png');
		background-size: 100% 100%;
		padding-left: 42rpx;
		margin-bottom: 30rpx;

		.titBox {
			padding: 47rpx 0;
			text-align: center;
			height: 100%;

			.tit1 {
				color: #FFEBD2;
				font-size: 34rpx;
				font-weight: 600;
			}

			.tit2 {
				color: #FFEBD2;
				font-size: 22rpx;
				margin: 10rpx 0 26rpx 0;
			}

			.tit3 {
				color: #FFDAAF;
				font-size: 24rpx;

				.iconfont {
					font-size: 20rpx;
				}
			}
		}

		.listBox {
			padding: 14rpx 0;

			.listActive {
				background-image: url('~@/static/images/lingyhj.png');
				background-size: 100% 100%;
			}

			.listHui {
				background-image: url('~@/static/images/weiling.png');
				background-size: 100% 100%;
			}

			.list {
				width: 170rpx;
				height: 210rpx;
				padding: 16rpx 0;
				text-align: center;
				margin-left: 24rpx;

				.tit {
					font-size: 18rpx;
					padding: 0 26rpx;
				}

				.titActive {
					color: #C99959;
				}

				.price {
					font-size: 46rpx;
					font-weight: 900;
					margin-top: 4rpx;
				}

				.pricehui {
					color: #B2B2B2;
				}

				.fonthui {
					background-color: #F5F5F5 !important;
				}

				.yuan {
					font-size: 24rpx;
				}

				.ling {
					font-size: 24rpx;
					margin-top: 9.5rpx;
					width: 102rpx;
					height: 36rpx;
					line-height: 36rpx;
					background-color: #FFE5C7;
					border-radius: 28rpx;
					margin: auto;
				}

				.priceM {
					color: #FFDAAF;
					font-size: 22rpx;
					margin-top: 14rpx;
				}
			}
		}
	}

	.sticky-box {
		/* #ifndef APP-PLUS-NVUE */
		display: flex;
		position: -webkit-sticky;
		/* #endif */
		position: sticky;
		/* #ifdef H5*/
		top: var(--window-top);
		/* #endif */
		z-index: 99;
		flex-direction: row;
		margin: 0px;
		background: #f5f5f5;
		padding: 30rpx 0;
	}

	.listAll {
		width: 20%;
		text-indent: 62rpx;
		font-size: 30rpx;
		border-left: 1px #eee solid;
		margin: 1% 0;
		padding: 5rpx;
		position: relative;

		image {
			position: absolute;
			left: 20rpx;
			top: 8rpx;
		}
	}

	.tab {
		position: relative;
		display: flex;
		font-size: 28rpx;
		white-space: nowrap;

		&__item {
			flex: 1;
			padding: 0 20rpx;
			text-align: center;
			height: 60rpx;
			line-height: 60rpx;
			color: #666;

			&.active {
				color: #09C2C9;
			}
		}
	}

	.tab__line {
		display: block;
		height: 6rpx;
		position: absolute;
		bottom: 0;
		left: 0;
		z-index: 1;
		border-radius: 3rpx;
		position: relative;
		background: #2FC6CD;
	}

	.scroll-view_H {
		/* 文本不会换行，文本会在在同一行上继续，直到遇到 <br> 标签为止。 */
		white-space: nowrap;
		width: 100%;
	}

	.page-index {
		display: flex;
		flex-direction: column;
		min-height: 100%;
		background: linear-gradient(180deg, #fff 0%, #f5f5f5 100%);

		.search {
			padding-left: 30rpx;
			background-color: #fff !important;
		}

		.search {
			padding-top: 20rpx;
			padding-bottom: 20rpx;
		}

		.search .input {
			width: 598rpx;
			background-color: #f7f7f7;
			border-radius: 33rpx;
			padding: 0 35rpx;
			box-sizing: border-box;
			height: 66rpx;
		}

		.search .input input {
			width: 472rpx;
			font-size: 26rpx;
		}

		.search .input .placeholder {
			color: #bbb;
		}

		.search .input .iconfont {
			color: #000;
			font-size: 35rpx;
		}

		.search .bnt {
			width: 120rpx;
			text-align: center;
			height: 66rpx;
			line-height: 66rpx;
			font-size: 30rpx;
			color: #282828;
		}

		.header {
			width: 100%;
			background-color: $theme-color;
			padding: 28rpx 30rpx;

			.serch-wrapper {
				align-items: center;

				.logo {
					width: 118rpx;
					height: 42rpx;
					margin-right: 24rpx;
				}

				image {
					width: 118rpx;
					height: 42rpx;
				}

				.input {
					display: flex;
					align-items: center;
					width: 546rpx;
					height: 58rpx;
					padding: 0 0 0 30rpx;
					background: rgba(247, 247, 247, 1);
					border: 1px solid rgba(241, 241, 241, 1);
					border-radius: 29rpx;
					color: #BBBBBB;
					font-size: 26rpx;

					.iconfont {
						margin-right: 20rpx;
						font-size: 26rpx;
						color: #666666;
					}
				}
			}

			.tabNav {
				padding-top: 24rpx;
			}
		}

		/* #ifdef MP */
		.mp-header {
			z-index: 999;
			position: fixed;
			left: 0;
			top: 0;
			width: 100%;
			/* #ifdef H5 */
			padding-bottom: 20rpx;
			/* #endif */
			background-color: $theme-color;

			.serch-wrapper {
				height: 100%;
				align-items: center;
				padding: 0 50rpx 0 53rpx;

				image {
					width: 118rpx;
					height: 42rpx;
					margin-right: 30rpx;
				}

				.input {
					display: flex;
					align-items: center;
					/* #ifdef MP */
					width: 305rpx;
					/* #endif */
					height: 50rpx;
					padding: 0 0 0 30rpx;
					background: rgba(247, 247, 247, 1);
					border: 1px solid rgba(241, 241, 241, 1);
					border-radius: 29rpx;
					color: #BBBBBB;
					font-size: 28rpx;

					.iconfont {
						margin-right: 20rpx;
					}
				}
			}
		}

		/* #endif */
		.page_content {
			background-color: #f5f5f5;
			/* #ifdef H5 */
			// margin-top: 20rpx !important;
			/* #endif */
			padding: 0 30rpx;

			.swiper {
				position: relative;
				width: 100%;
				height: 280rpx;
				margin: 0 auto;
				border-radius: 10rpx;
				overflow: hidden;
				margin-bottom: 25rpx;
				/* #ifdef MP */
				z-index: 10;
				margin-top: 20rpx;

				/* #endif */
				swiper,
				.swiper-item,
				image {
					width: 100%;
					height: 280rpx;
					border-radius: 10rpx;
				}
			}

			.nav {
				padding-bottom: 26rpx;
				background: #fff;
				opacity: 1;
				border-radius: 14rpx;
				width: 100%;
				margin-bottom: 30rpx;

				.item {
					display: flex;
					flex-direction: column;
					align-items: center;
					justify-content: center;
					width: 25%;
					margin-top: 30rpx;

					image {
						width: 82rpx;
						height: 82rpx;
					}
				}
			}

			.nav-bd {
				display: flex;
				align-items: center;
				justify-content: space-between;

				.item {
					display: flex;
					flex-direction: column;
					align-items: center;
					justify-content: center;

					.txt {
						font-size: 32rpx;
						color: #282828;
					}

					.label {
						display: flex;
						align-items: center;
						justify-content: center;
						width: 124rpx;
						height: 32rpx;
						margin-top: 5rpx;
						font-size: 24rpx;
						color: #999;
					}

					&.active {
						color: $theme-color;

						.txt {
							color: $theme-color;
						}

						.label {
							background: linear-gradient(90deg, $bg-star 0%, $bg-end 100%);
							border-radius: 16rpx;
							color: #fff;
						}
					}
				}
			}

			.index-product-wrapper {
				margin-bottom: 110rpx;

				&.on {
					min-height: 1500rpx;
				}

				.list-box {
					display: flex;
					flex-wrap: wrap;
					justify-content: space-between;

					.item {
						width: 335rpx;
						margin-bottom: 20rpx;
						background-color: #fff;
						border-radius: 10rpx;
						overflow: hidden;

						image {
							width: 100%;
							height: 430rpx;
						}

						.text-info {
							padding: 10rpx 20rpx 15rpx;

							.title {
								color: #222222;
							}

							.old-price {
								margin-top: 8rpx;
								font-size: 26rpx;
								color: #AAAAAA;
								text-decoration: line-through;

								text {
									margin-right: 2px;
									font-size: 20rpx;
								}
							}

							.price {
								display: flex;
								align-items: flex-end;
								color: $theme-color;
								font-size: 34rpx;
								font-weight: 800;

								text {
									padding-bottom: 4rpx;
									font-size: 24rpx;
									font-weight: 800;
								}

								.txt {
									display: flex;
									align-items: center;
									justify-content: center;
									width: 28rpx;
									height: 28rpx;
									margin-left: 15rpx;
									margin-bottom: 10rpx;
									border: 1px solid $theme-color;
									border-radius: 4rpx;
									font-size: 22rpx;
									font-weight: normal;
								}
							}
						}
					}

					&.on {
						display: flex;
					}
				}
			}
		}
	}

	.pictrue {
		position: relative;
	}

	.fixed {
		z-index: 100;
		position: fixed;
		left: 0;
		top: 0;
		background: linear-gradient(90deg, red 50%, #ff5400 100%);
	}

	.mores-txt {
		width: 100%;
		align-items: center;
		justify-content: center;
		height: 70rpx;
		color: #999;
		font-size: 24rpx;

		.iconfont {
			margin-top: 2rpx;
			font-size: 20rpx;
		}
	}

	.menu-txt {
		font-size: 24rpx;
		color: #454545;
	}

	.mp-bg {
		position: absolute;
		left: 0;
		/* #ifdef H5 */
		top: 98rpx;
		/* #endif */
		width: 100%;
		height: 304rpx;
		background: linear-gradient(180deg, #E93323 0%, #F5F5F5 100%, #751A12 100%);
		// border-radius: 0 0 30rpx 30rpx;
	}
</style>
