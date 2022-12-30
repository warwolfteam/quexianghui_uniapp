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
				</view>
				<view class="collection">
					<view class="collection_title flex justify-between color_000 fontS32 mar_top30">
						<text>我解锁的资料</text>
						<view class="select_box flex align-center color_828282 fontS24"
							@click="goAllCollection">
							<text>全部</text>
							<u-icon name="arrow-right"
								:size="16"
								color="#828282"></u-icon>
						</view>
					</view>
					<scroll-view :scroll-x="true"
						@scrolltolower="collectionRight">
						<view class="flex">
							<view v-for="(item, index) in list"
								:key="index"
								class="collectionList"
								@click="goDetails(item, 0)">
								<image :src="`${osshttp}${item.detail.thumb_image}`"></image>
							</view>
						</view>
					</scroll-view>
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
					"name": "开通会员",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/dd497d333f4541d7ae41b2c579706370jhp2x39yrg.png",
					"id": 1238,
					"url": "/pages/users/user_vip/index"
				}, {
					"wap_url": "/pages/users/user_money/index",
					"name": "我的余额",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/0624fe22d770438891ad666ebef9e21e1rhiepyec1.png",
					"id": 1241,
					"url": "/pages/users/user_money/index"
				}, {
					"wap_url": "/pages/users/order_list/index",
					"name": "我的订单",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/886e04146694474c966d346222fe2897ealwo6qycj.png",
					"id": 1242,
					"url": "/pages/users/order_list/index"
				}, {
					"wap_url": "/pages/message/message",
					"name": "消息通知",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/886e04146694474c966d346222fe2897ealwo6qycj.png",
					"id": 1242,
					"url": "/pages/message/message"
				}, {
					"wap_url": "/pages/xiangqin/user_detail/user_detail",
					"name": "我的资料",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/11113e1876ac427bb3173086a3bd6c5bhyan1ea9pz.png",
					"id": 1244,
					"url": "/pages/xiangqin/user_detail/user_detail"
				}, {
					"wap_url": "/pages/users/user_goods_collection/index",
					"name": "我的收藏",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/11113e1876ac427bb3173086a3bd6c5bhyan1ea9pz.png",
					"id": 1244,
					"url": "/pages/users/user_goods_collection/index"
				}, {
					"wap_url": "/pages/RealNameAuthent/RealNameAuthent",
					"name": "身份认证",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/11113e1876ac427bb3173086a3bd6c5bhyan1ea9pz.png",
					"id": 1244,
					"url": "/pages/RealNameAuthent/RealNameAuthent"
				}, {
					"wap_url": "/pages/users/user_address_list/index",
					"name": "申请商家",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/11113e1876ac427bb3173086a3bd6c5bhyan1ea9pz.png",
					"id": 1244,
					"url": "/pages/users/user_address_list/index"
				}, {
					"wap_url": "https://yzf.qq.com/xv/web/static/chat/index.html?sign=37ef9b97db2656c32340cde61ce2b56a2176efe72ac7ed421c77607b5c816611ec4775a17c7605b33df1ffe1d22a4ce7464dd07b",
					"name": "联系客服",
					"pic": "https://api.centosxyc1.qqfrp.heimaoba.cn/crmebimage/public/maintain/2021/12/25/77ac54ce5f514ebbb89854f927f2a891dzefr2qxmh.png",
					"id": 1245,
					"url": "/pages/service/index"
				}],
				osshttp: "https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/",
				list: [{
					"goods_id": 8,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 8,
						"name": "模型测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221018/377ebd29f9a83d44a5cf2168a86e15f0.gif",
						"thumb_image": "/uploads/20221018/377ebd29f9a83d44a5cf2168a86e15f0.gif",
						"main_obj": "/uploads/20221018/11dcf7a95499f884555fa07d6bb6e5e3.glb",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 3,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1666080900,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p style=\"text-align: center; \"><iframe webkitallowfullscreen=\"\" mozallowfullscreen=\"\" allowfullscreen=\"\" frameborder=\"0\" height=\"310\" width=\"500\" src=\"http://v.qq.com/iframe/player.html?vid=t3360xmtq6t&amp;auto=0\" class=\"note-video-clip\"></iframe><br></p>",
						"compound_start_time": 1666080903,
						"compound_end_time": 1666080903,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1666080972,
						"updatetime": 1667284274,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 56,
							"user_id": 230145,
							"order_id": 41,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 8,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667185314,
							"updatetime": 1667185321,
							"task_id": "20221031110201ajhs05ZrQl",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-10-18 16:15:00",
						"sale_status_text": 0,
						"sale_total_num_set": 3
					}
				}, {
					"goods_id": 10,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 10,
						"name": "上链测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221019/efab49617a0df451a2bb3bde67b80d52.png",
						"thumb_image": "/uploads/20221019/efab49617a0df451a2bb3bde67b80d52.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 21,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1666166640,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><br></p>",
						"compound_start_time": 1666166674,
						"compound_end_time": 1666166674,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1666166768,
						"updatetime": 1669260915,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 53,
							"user_id": 230145,
							"order_id": 39,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 10,
							"collection_code": "#020/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1666950961,
							"updatetime": 1671040486,
							"task_id": "20221028175602w148v3Iljc",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-10-19 16:04:00",
						"sale_status_text": 0,
						"sale_total_num_set": 21
					}
				}, {
					"goods_id": 12,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 12,
						"name": "模型",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 3,
						"types": 1,
						"main_image": "/uploads/20221024/0c036ead8e32e001d6f47b96d7a1440a.jpg",
						"thumb_image": "/uploads/20221024/0c036ead8e32e001d6f47b96d7a1440a.jpg",
						"main_obj": "/uploads/20221024/ebbe4192a4c0a526254a4b6dc8bbb888.glb",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 15,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1666593540,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p>123456</p>",
						"compound_start_time": 1666593568,
						"compound_end_time": 1666593568,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1666593655,
						"updatetime": 1667554393,
						"deleted": 0,
						"purchasing_num": 1,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 52,
							"user_id": 230145,
							"order_id": 38,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 12,
							"collection_code": "#015/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1666866981,
							"updatetime": 1666867021,
							"task_id": "20221027183701pag2D4KSAL",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-10-24 14:39:00",
						"sale_status_text": 0,
						"sale_total_num_set": 15
					}
				}, {
					"goods_id": 15,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 15,
						"name": "正方形猫",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 2,
						"main_image": "/uploads/20221101/2aea6ab42c0a953ca1e31f7e3234c6e6.jpg",
						"thumb_image": "/uploads/20221101/2aea6ab42c0a953ca1e31f7e3234c6e6.jpg",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 1,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667269140,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/996e38997b6d45d326f34429ecc344f4.png\" style=\"width: 794px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667269170,
						"compound_end_time": 1669774770,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667269321,
						"updatetime": 1667289138,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 64,
							"user_id": 230145,
							"order_id": 49,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 15,
							"collection_code": "#001/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667285092,
							"updatetime": 1667285102,
							"task_id": "20221101144502dJucUfH1r4",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 10:19:00",
						"sale_status_text": 0,
						"sale_total_num_set": 2
					}
				}, {
					"goods_id": 17,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 17,
						"name": "徽章3D藏品  测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 3,
						"types": 2,
						"main_image": "/uploads/20221101/4d964fc627e1e9afb02363249691fdaf.gif",
						"thumb_image": "/uploads/20221101/4d964fc627e1e9afb02363249691fdaf.gif",
						"main_obj": "/uploads/20221101/ebbe4192a4c0a526254a4b6dc8bbb888.glb",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 6,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667269680,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/2aea6ab42c0a953ca1e31f7e3234c6e6.jpg\" style=\"width: 941px;\" data-filename=\"filename\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/7051b7fe8942ebbe939ad04e8e479a58.jpg\" style=\"width: 941px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667269717,
						"compound_end_time": 1669775317,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667269810,
						"updatetime": 1667382849,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 63,
							"user_id": 230145,
							"order_id": 48,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 17,
							"collection_code": "#006/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667284752,
							"updatetime": 1667284801,
							"task_id": "20221101144001wvQUTN92Bp",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 10:28:00",
						"sale_status_text": 0,
						"sale_total_num_set": 5
					}
				}, {
					"goods_id": 19,
					"goods_type": 2,
					"count": 6,
					"detail": {
						"id": 19,
						"name": "3D蜜蜂",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png",
						"thumb_image": "/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png",
						"main_obj": "/uploads/20221101/06bf1ece41818483a027ef937b8f7c7c.glb",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 8,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667287860,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png\" style=\"width: 717px;\" data-filename=\"filename\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png\" data-filename=\"filename\" style=\"width: 717px;\"><br></p>",
						"compound_start_time": 1667287871,
						"compound_end_time": 1667287871,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667287950,
						"updatetime": 1669260930,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 208,
							"user_id": 230145,
							"order_id": 0,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 19,
							"collection_code": "#008/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 3,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1667958058,
							"updatetime": 1667958058,
							"task_id": null,
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 15:31:00",
						"sale_status_text": 0,
						"sale_total_num_set": 8
					}
				}, {
					"goods_id": 20,
					"goods_type": 2,
					"count": 4,
					"detail": {
						"id": 20,
						"name": "需要合成1",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/f5079e7d2c02ea8d56dae030a973dfb0.png",
						"thumb_image": "/uploads/20221101/f5079e7d2c02ea8d56dae030a973dfb0.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 12,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667288580,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/f5079e7d2c02ea8d56dae030a973dfb0.png\" data-filename=\"filename\" style=\"width: 717px;\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/f5079e7d2c02ea8d56dae030a973dfb0.png\" style=\"width: 717px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667288591,
						"compound_end_time": 1667288591,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667288615,
						"updatetime": 1669260927,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 99,
							"user_id": 230145,
							"order_id": 63,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 20,
							"collection_code": "#005/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667297729,
							"updatetime": 1667297763,
							"task_id": "20221101181603wc4ofDKJaP",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 15:43:00",
						"sale_status_text": 0,
						"sale_total_num_set": 12
					}
				}, {
					"goods_id": 21,
					"goods_type": 2,
					"count": 4,
					"detail": {
						"id": 21,
						"name": "需要合成2",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/23d49c92e5d84495da5af0e090f6b948.png",
						"thumb_image": "/uploads/20221101/23d49c92e5d84495da5af0e090f6b948.png",
						"main_obj": "",
						"total_num": 97,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 8,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667288640,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/23d49c92e5d84495da5af0e090f6b948.png\" data-filename=\"filename\" style=\"width: 717px;\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/23d49c92e5d84495da5af0e090f6b948.png\" style=\"width: 717px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667288668,
						"compound_end_time": 1667288668,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667288707,
						"updatetime": 1669260929,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 94,
							"user_id": 230145,
							"order_id": 62,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 21,
							"collection_code": "#04/97",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667297718,
							"updatetime": 1667297762,
							"task_id": "20221101181602Oj4RTEV75a",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 15:44:00",
						"sale_status_text": 0,
						"sale_total_num_set": 8
					}
				}, {
					"goods_id": 25,
					"goods_type": 2,
					"count": 9,
					"detail": {
						"id": 25,
						"name": "爱莲说",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/a9fbc9ea73ac417d1347624b0839fa74.jpeg",
						"thumb_image": "/uploads/20221101/a9fbc9ea73ac417d1347624b0839fa74.jpeg",
						"main_obj": "",
						"total_num": 999,
						"prev_total_num": 0,
						"price": "100.00",
						"sale_total_num": 20,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667289600,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/a9fbc9ea73ac417d1347624b0839fa74.jpeg\" style=\"width: 400px;\"><br></p>",
						"compound_start_time": 1667289656,
						"compound_end_time": 1667289656,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667289921,
						"updatetime": 1669260939,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 76,
							"user_id": 230145,
							"order_id": null,
							"order_type": 3,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 25,
							"collection_code": "#009/999",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 5,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1667290388,
							"updatetime": 1667290443,
							"task_id": "202211011614033oFEdby60T",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 16:00:00",
						"sale_status_text": 0,
						"sale_total_num_set": 20
					}
				}, {
					"goods_id": 26,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 26,
						"name": "玫瑰少年",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 2,
						"main_image": "/uploads/20221101/03bcf903d1a145a9c07a56a82d979cc6.webp",
						"thumb_image": "/uploads/20221101/03bcf903d1a145a9c07a56a82d979cc6.webp",
						"main_obj": "",
						"total_num": 999,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 1,
						"sale_total_price": "0.00",
						"selling_state": 1,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667291580,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "",
						"compound_start_time": 1667291601,
						"compound_end_time": 1668501201,
						"user_synthesis_times": 1,
						"compound_nums": 1,
						"createtime": 1667291755,
						"updatetime": 1669260937,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 91,
							"user_id": 230145,
							"order_id": null,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 26,
							"collection_code": "#001/999",
							"status": 1,
							"types": 3,
							"for_sale": 1,
							"source_type": 6,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1667297515,
							"updatetime": 1667297515,
							"task_id": null,
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 16:33:00",
						"sale_status_text": 0,
						"sale_total_num_set": 0
					}
				}, {
					"goods_id": 28,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 28,
						"name": "盲盒 藏品 1",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png",
						"thumb_image": "/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 2,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667293740,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png\" style=\"width: 717px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667293788,
						"compound_end_time": 1667293788,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667293816,
						"updatetime": 1669260932,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 82,
							"user_id": 230145,
							"order_id": 0,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 28,
							"collection_code": "#001/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 3,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1667294942,
							"updatetime": 1667294942,
							"task_id": null,
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 17:09:00",
						"sale_status_text": 0,
						"sale_total_num_set": 2
					}
				}, {
					"goods_id": 29,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 29,
						"name": "盲盒 藏品 2",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/874e384f427945086044cdb02f6ef6d4.png",
						"thumb_image": "/uploads/20221101/874e384f427945086044cdb02f6ef6d4.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 2,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667293800,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/874e384f427945086044cdb02f6ef6d4.png\" data-filename=\"filename\" style=\"width: 717px;\"><br></p>",
						"compound_start_time": 1667293850,
						"compound_end_time": 1667293850,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667293882,
						"updatetime": 1669260935,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 87,
							"user_id": 230145,
							"order_id": 0,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 29,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 3,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1667296802,
							"updatetime": 1667296802,
							"task_id": null,
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 17:10:00",
						"sale_status_text": 0,
						"sale_total_num_set": 2
					}
				}, {
					"goods_id": 32,
					"goods_type": 2,
					"count": 2,
					"detail": {
						"id": 32,
						"name": "合成--得的藏品",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 2,
						"main_image": "/uploads/20221101/346f78ed546a49b73dba4e22a75e2576.png",
						"thumb_image": "/uploads/20221101/346f78ed546a49b73dba4e22a75e2576.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 3,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667297520,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221101/cb2587a1801a48d9da0d44b190809b73.png\" style=\"width: 692px;\" data-filename=\"filename\"><br></p>",
						"compound_start_time": 1667297550,
						"compound_end_time": 1669803150,
						"user_synthesis_times": 100,
						"compound_nums": 3,
						"createtime": 1667297597,
						"updatetime": 1669260947,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 244,
							"user_id": 230145,
							"order_id": null,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 32,
							"collection_code": "#003/100",
							"status": 1,
							"types": 3,
							"for_sale": 1,
							"source_type": 6,
							"from_user_id": 0,
							"from_user_collection_id": null,
							"createtime": 1668245762,
							"updatetime": 1668245762,
							"task_id": null,
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-01 18:12:00",
						"sale_status_text": 0,
						"sale_total_num_set": 0
					}
				}, {
					"goods_id": 35,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 35,
						"name": "藏品 11.2--测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/2aea6ab42c0a953ca1e31f7e3234c6e6.jpg",
						"thumb_image": "/uploads/20221101/2aea6ab42c0a953ca1e31f7e3234c6e6.jpg",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 3,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667381520,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/1fcc50973814ceb47424025beb3d269f.jpg\" style=\"width: 941px;\"><br></p>",
						"compound_start_time": 1667381563,
						"compound_end_time": 1667381563,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667381694,
						"updatetime": 1667467985,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 3,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 107,
							"user_id": 230145,
							"order_id": 66,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 35,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667382113,
							"updatetime": 1667382121,
							"task_id": "202211021742015Hf3SpB0Ki",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-02 17:32:00",
						"sale_status_text": 0,
						"sale_total_num_set": 3
					}
				}, {
					"goods_id": 38,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 38,
						"name": "盲盒藏品3__11.2",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221023/479bfcfaf71c64682d1bd76c86ede04c.jpg",
						"thumb_image": "/uploads/20221023/479bfcfaf71c64682d1bd76c86ede04c.jpg",
						"main_obj": "",
						"total_num": 60,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 3,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667382600,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/49944ced5d1ea1753d8b954490128661.jpg\" data-filename=\"filename\" style=\"width: 748px;\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/222f75697d42b6b34e81248360978fa5.jpg\" data-filename=\"filename\" style=\"width: 941px;\"><br></p>",
						"compound_start_time": 1667382636,
						"compound_end_time": 1667382636,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667382744,
						"updatetime": 1667637085,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 111,
							"user_id": 230145,
							"order_id": 69,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 38,
							"collection_code": "#03/60",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667383285,
							"updatetime": 1667383322,
							"task_id": "20221102180202Vg5XBWEwmh",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-02 17:50:00",
						"sale_status_text": 0,
						"sale_total_num_set": 3
					}
				}, {
					"goods_id": 39,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 39,
						"name": "藏品 11.2-- 2 测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221101/241f5acb2bad7fac43b9e5a2e9ee9d20.jpg",
						"thumb_image": "/uploads/20221101/241f5acb2bad7fac43b9e5a2e9ee9d20.jpg",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "0.00",
						"sale_total_num": 3,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667384340,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/2148c5ab508aa6b7469306dbae5e72cb.jpg\" style=\"width: 941px;\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/d38858ea76f71c1a359a404bd8810fe8.png\" style=\"width: 532px;\"><br></p>",
						"compound_start_time": 1667384390,
						"compound_end_time": 1667384390,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667384804,
						"updatetime": 1667467987,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 4,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 114,
							"user_id": 230145,
							"order_id": 73,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 39,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667461764,
							"updatetime": 1667461801,
							"task_id": "20221103155001UfJVeQlaY8",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-02 18:19:00",
						"sale_status_text": 0,
						"sale_total_num_set": 3
					}
				}, {
					"goods_id": 40,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 40,
						"name": "藏品 11.3--测试",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221103/a9a307c83d80a235b6911bb45a51797b.png",
						"thumb_image": "/uploads/20221103/a9a307c83d80a235b6911bb45a51797b.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "1.00",
						"sale_total_num": 2,
						"sale_total_price": "0.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 0,
						"compound_num": 1,
						"release_time": 1667466120,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221103/2148c5ab508aa6b7469306dbae5e72cb.jpg\" style=\"width: 905px;\"><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/2148c5ab508aa6b7469306dbae5e72cb.jpg\" style=\"width: 905px;\"><br></p>",
						"compound_start_time": 1667466176,
						"compound_end_time": 1667466176,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667466244,
						"updatetime": 1671039939,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 4,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 126,
							"user_id": 230145,
							"order_id": 81,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 40,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667470299,
							"updatetime": 1667470322,
							"task_id": "20221103181201X7WsmS2R83",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-03 17:02:00",
						"sale_status_text": 0,
						"sale_total_num_set": 1
					}
				}, {
					"goods_id": 47,
					"goods_type": 2,
					"count": 2,
					"detail": {
						"id": 47,
						"name": "藏品 11.3--测试2",
						"issuer_id": 0,
						"series_id": 1,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221102/49944ced5d1ea1753d8b954490128661.jpg",
						"thumb_image": "/uploads/20221102/49944ced5d1ea1753d8b954490128661.jpg",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "1.00",
						"sale_total_num": 2,
						"sale_total_price": "2.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 0,
						"tmp_gift_flag": 0,
						"select_status": 0,
						"show_index": 1,
						"compound_num": 1,
						"release_time": 1667468520,
						"preemption_num": 0,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221102/49944ced5d1ea1753d8b954490128661.jpg\" style=\"width: 748px;\"><br></p>",
						"compound_start_time": 1667468577,
						"compound_end_time": 1667468577,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667468614,
						"updatetime": 1671039929,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 0,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 133,
							"user_id": 230145,
							"order_id": 92,
							"order_type": 1,
							"series_id": 1,
							"blind_box_id": null,
							"collection_id": 47,
							"collection_code": "#002/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 1,
							"from_user_id": 0,
							"from_user_collection_id": 0,
							"createtime": 1667632604,
							"updatetime": 1667632621,
							"task_id": "20221105151701QZocfuRGeM",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-03 17:42:00",
						"sale_status_text": 0,
						"sale_total_num_set": 2
					}
				}, {
					"goods_id": 50,
					"goods_type": 2,
					"count": 2,
					"detail": {
						"id": 50,
						"name": "藏品 11.5--测试2",
						"issuer_id": 0,
						"series_id": 22,
						"display_type": 2,
						"types": 1,
						"main_image": "/uploads/20221105/04ba824f5b3e69816b0f4681d722c046.jpg",
						"thumb_image": "/uploads/20221105/04ba824f5b3e69816b0f4681d722c046.jpg",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "1.00",
						"sale_total_num": 2,
						"sale_total_price": "1.00",
						"selling_state": 0,
						"for_sale": 1,
						"tmp_sell_flag": 1,
						"tmp_gift_flag": 1,
						"select_status": 0,
						"show_index": 1,
						"compound_num": 1,
						"release_time": 1667643180,
						"preemption_num": 2,
						"ant_chain": "",
						"introduce_image": "<p><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221105/04ba824f5b3e69816b0f4681d722c046.jpg\" style=\"width: 690px;\"><span style=\"font-size: 12px;\">阿斯顿发阿斯顿发123.</span><br></p>",
						"compound_start_time": 1667643217,
						"compound_end_time": 1667643217,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667643303,
						"updatetime": 1671039928,
						"deleted": 0,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 2,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 1,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 205,
							"user_id": 230145,
							"order_id": 8,
							"order_type": 2,
							"series_id": 21,
							"blind_box_id": null,
							"collection_id": 50,
							"collection_code": "#001/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 2,
							"from_user_id": 230164,
							"from_user_collection_id": 135,
							"createtime": 1667901376,
							"updatetime": 1667901421,
							"task_id": "202211081757016gV0LFysid",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-05 18:13:00",
						"sale_status_text": 0,
						"sale_total_num_set": 0
					}
				}, {
					"goods_id": 53,
					"goods_type": 2,
					"count": 1,
					"detail": {
						"id": 53,
						"name": "盲盒藏品1_11.5",
						"issuer_id": 0,
						"series_id": 23,
						"display_type": 1,
						"types": 1,
						"main_image": "/uploads/20221103/a9a307c83d80a235b6911bb45a51797b.png",
						"thumb_image": "/uploads/20221103/a9a307c83d80a235b6911bb45a51797b.png",
						"main_obj": "",
						"total_num": 100,
						"prev_total_num": 0,
						"price": "1.00",
						"sale_total_num": 12,
						"sale_total_price": "2.00",
						"selling_state": 1,
						"for_sale": 1,
						"tmp_sell_flag": 1,
						"tmp_gift_flag": 1,
						"select_status": 0,
						"show_index": 1,
						"compound_num": 1,
						"release_time": 1667648280,
						"preemption_num": 1,
						"ant_chain": "",
						"introduce_image": "<p><span style=\"font-size: 12px; font-family: 微软雅黑;\">阿萨德了咖啡馆里看见</span><img src=\"https://heyinshucang.oss-cn-hangzhou.aliyuncs.com/uploads/20221103/a9a307c83d80a235b6911bb45a51797b.png\" style=\"width: 675px;\"><br></p>",
						"compound_start_time": 1667648301,
						"compound_end_time": 1667648301,
						"user_synthesis_times": 1,
						"compound_nums": 0,
						"createtime": 1667648409,
						"updatetime": 1669260977,
						"deleted": 1,
						"purchasing_num": 0,
						"open_draw": 0,
						"success_rate": 0,
						"collection_grade": 0,
						"resale_time": 0,
						"white_mobiles": "",
						"position_collection_id": "",
						"position_blindbox_id": 0,
						"pay_start_time": null,
						"is_preemption": 1,
						"shou_num": 0,
						"pay_num": 0,
						"kou_num": 0,
						"is_hua": 0,
						"pay_wallet": 0,
						"currency_id": 0,
						"one_user_collection": {
							"id": 204,
							"user_id": 230145,
							"order_id": 7,
							"order_type": 2,
							"series_id": 23,
							"blind_box_id": null,
							"collection_id": 53,
							"collection_code": "#003/100",
							"status": 1,
							"types": 2,
							"for_sale": 1,
							"source_type": 2,
							"from_user_id": 230164,
							"from_user_collection_id": 136,
							"createtime": 1667900534,
							"updatetime": 1667900582,
							"task_id": "202211081743027MB3sCbl6e",
							"node_id": null,
							"to_gather_time": 0,
							"from_gather_time": 0,
							"remarks": null,
							"re_task": 0,
							"operation_id": "",
							"number": 0,
							"durable": 0,
							"strengthen": 0,
							"enchanting": 0,
							"power": "0.00",
							"explosive": "0.00",
							"max_power": "0.00",
							"max_explosive": "0.00",
							"max_durable": 0,
							"create_node_id": 0
						},
						"release_time_text": 0,
						"release_time_date_text": "2022-11-05 19:38:00",
						"sale_status_text": 0,
						"sale_total_num_set": 12
					}
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
			// 全部资料跳转
			goAllCollection() {
				console.log("全部资料跳转");
				uni.navigateTo({
					url: '/pages/xiangqin/all_users/all_users'
				})
			},
			// 资料翻页
			collectionRight() {
				console.log("// 资料翻页");
			},
			// 跳转详情
			goDetails(item, type) {
				console.log(item, type);
				if (item.activityH5 && item.activityH5.type === "2" && !this.isLogin) {
					toLogin();
				} else {
					uni.navigateTo({
						url: `/pages/xiangqin/user_detail/user_detail?item=` + encodeURIComponent(JSON.stringify(
							item))
					})
				}
			},
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
