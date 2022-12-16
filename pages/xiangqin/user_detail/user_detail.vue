<template>
	<view class="user_detail">
		<view class="user_detail_head">
			<view class="header_box">
				<u-image width="100%"
					height="600rpx"
					border-radius="30rpx"
					mode="aspectFit"
					:src="userInfo.avatar"></u-image>
			</view>
		</view>
		<view class="my_body">
			<u-cell-group title="个人信息">
				<u-cell-item title="昵称"
					value="妙儿"></u-cell-item>
				<u-cell-item title="性别"
					value="男"></u-cell-item>
				<u-cell-item title="区域"
					value="冰岛"></u-cell-item>
				<u-cell-item title="年龄"
					value="8"></u-cell-item>
			</u-cell-group>
		</view>
		<view class="simi_body">
			<u-cell-group title="联系方式">
				<u-cell-item title="微信号"
					value="123456"></u-cell-item>
				<u-cell-item title="手机号"
					value="13256458965"></u-cell-item>
			</u-cell-group>
		</view>
		<view class="zhanwei">
			
		</view>
	</view>
</template>
<script>
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
		},
		computed: mapGetters(['isLogin', 'uid', 'userInfo']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getCityList();
					}
				},
				deep: true
			}
		},
		data() {
			return {
				showSetAge: false,
				showSetSex: false,
				ageList: [],
				sexList: [{
					value: '1',
					label: '男'
				}, {
					value: '2',
					label: '女'
				}],
				regionDval: ['浙江省', '杭州市', '滨江区'],
				id: 0, //地址id
				region: ['省', '市', '区'],
				valueRegion: [0, 0, 0],
				district: [],
				multiArray: [],
				multiIndex: [0, 0, 0],
				cityId: 0,
				defaultRegion: ['广东省', '广州市', '番禺区'],
				defaultRegionCode: '440113',
				memberInfo: {},
				loginType: 'h5', //app.globalData.loginType
				userIndex: 0,
				newAvatar: '',
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				diqu: "江西-赣州"
			}
		},
		onLoad() {
			if (!this.isLogin) {
				toLogin();
			} else {
				console.log("userInfo", this.userInfo);
			}
		},
		methods: {}
	}
</script>
<style scoped
	lang="scss">
	.user_detail {
		background-color: #f5f5f5;
		width: 100%;

		.user_detail_head {
			padding: 15px;

			.header_box {
				padding: 15px;
				width: 100%;
				border-radius: 30rpx;
				text-align: center;
				background: #ffffff;
			}
		}

		.my_body {
			margin: 30rpx;
			padding: 15px;
			border-radius: 30rpx;
			text-align: center;
			background: #ffffff;
		}

		.simi_body {
			margin: 30rpx;
			padding: 15px;
			border-radius: 30rpx;
			text-align: center;
			background: #ffffff;
		}
		.zhanwei{
			height: 100rpx;
		}
	}
</style>
