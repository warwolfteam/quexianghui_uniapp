<template>
	<view class="user_detail">
		<view class="user_detail_head">
			<view class="header_box">
				<u-image width="100%" height="600rpx" border-radius="30rpx" mode="aspectFit" :src="headImg">
				</u-image>
			</view>
		</view>
		<view class="simi_body">
			<view class="nobuy" v-if="jiesuoStatus==0">
				<view class="nobuy_detail detail_blur">
					<view class="nobuy_list">
						<view class="jiamitex">
							<view>联系方式: </view>
							<view class="jiami"> （信息已保密）</view>
						</view>
						<view class="jiamitex">手机号码：*** **** ****</view>
					</view>
					<view class="anniu"> 您还未解锁该联系方式！ </view>
					<u-button :custom-style="customStyle" @click="jiesuoziliao()" size="medium">立即解锁</u-button>
				</view>
			</view>
			<view class="buy" v-else>
				<u-cell-group title="联系方式">
					<u-cell-item title="微信号" :arrow="false" value="123456"></u-cell-item>
					<u-cell-item title="手机号" :arrow="false" value="13256458965"></u-cell-item>
				</u-cell-group>
			</view>
		</view>
		<view class="my_body">
			<u-cell-group title="用户信息">
				<u-cell-item title="昵称" :arrow="false" :value="nickName"></u-cell-item>
				<u-cell-item title="性别" :arrow="false" :value="sex"></u-cell-item>
				<u-cell-item title="区域" :arrow="false" :value="city"></u-cell-item>
				<u-cell-item title="年龄" :arrow="false" :value="age"></u-cell-item>
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
				headImg: "",
				nickName: "妙儿",
				sex: "男",
				city: "冰岛",
				age: "8",
				jiesuoStatus: 0,
				customStyle: {
					marginTop: '-20rpx', // 注意驼峰命名，并且值必须用引号包括，因为这是对象
					color: '#ff0053'
				},
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
		onLoad: function(option) {
			console.log("option0", option.id)
			console.log("option-item", option.item)

			if (!this.isLogin) {
				toLogin();
			}
			if (this.isLogin) {
				console.log("userInfo", this.userInfo);
				if (option.id === undefined) {
					console.log("option2", option.id);
					this.headImg = this.userInfo.avatar;
					this.jiesuoStatus = 1;
					if (option.item !== undefined) {
						var item = JSON.parse(decodeURIComponent(option.item));
						console.log("item", item);
						this.headImg = item.image;
						this.nickName = item.nickName;
						this.age = item.age;
						this.sex = "女";

						this.jiesuoStatus = 0;
					}
					if (option.item === undefined) {
						console.log("option.item", option.item);
					}
				}
				if (option.id !== undefined) {
					console.log("option1", option.id);

				}
			}
		},
		methods: {
			jiesuoziliao() {
				console.log("点击解锁资料");
				this.jiesuoStatus = 1;
			}
		}
	}
</script>
<style scoped lang="scss">
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

			.nobuy {
				width: 100%;

				.nobuy_detail {
					border-radius: 10rpx;
					padding: 50rpx;
					position: relative;
					border: rgba(255, 102, 138, .08) 1px solid;
					background-color: rgba(255, 102, 138, .02);

					.nobuy_list {
						line-height: 3rem;
						font-size: 13rpx;

						.jiamitex {
							display: flex;
							font-size: 13rpx;

							.jiami {
								color: #FD9F23;
								font-size: 26rpx;
							}
						}
					}

					.anniu {
						width: 100%;
						position: absolute;
						top: 0;
						left: 0;
						line-height: 230rpx;
						height: 170rpx;
						z-index: 1;
						text-align: center;
					}
				}

				.detail_blur .nobuy_list {
					filter: blur(.4rem);
					filter: alpha(opacity=50);
					-moz-opacity: 0.5;
					opacity: 0.5
				}
			}
		}

		.zhanwei {
			height: 100rpx;
		}
	}
</style>
