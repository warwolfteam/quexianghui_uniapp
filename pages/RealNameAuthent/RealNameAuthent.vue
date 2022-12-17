<template>
	<view class="">
		<view v-if="loading"
			class="">
			<z-loading height="90vh"></z-loading>
		</view>
		<view v-else
			class="">
			<view class="hander_box flex align-center"
				:style="{backgroundImage:`url(${backgroundImage})`}">
				<view v-if="!isOk"
					class="color_fff mar_left40 fontS48">填写实名认证信息</view>
				<view v-if="isOk"
					class="color_fff mar_left40 fontS48 flex align-center">
					<view class="mar_right10">已实名认证</view>
					<u-image mode="aspectFit"
						:src="okImgUrl"
						width="34.81rpx"
						height="44.49rpx"></u-image>
				</view>
			</view>
			<view class="my_info_box">
				<view class="height_85"></view>
				<view v-if="!isOk"
					class="info_box">
					<view class="color_828282 fontS26 bold mar_top60">真实姓名</view>
					<z-input maxlength="16"
						v-model="formData.user_name"
						textAlign="left"
						placeholder="请输入真实姓名"
						border="1px solid #E0E0E0"></z-input>
					<view class="color_828282 fontS26 bold mar_top30">身份证号</view>
					<z-input v-model="formData.id_num"
						maxlength="18"
						textAlign="left"
						placeholder="请输入18位身份证号"
						border="1px solid #E0E0E0"></z-input>
					<view class="flex align-center justify-center mar_top40">
						<u-icon :name="yesImgUrl"></u-icon>
						<text class="fontS20 color_828282 mar_left10">信息安全保障中，仅用于身份认证</text>
					</view>
					<view @click="submit"
						style="z-index: 999;"
						class="submit bgColorjb flex align-center justify-center mar_top40 fontS28 border_radius_20">
						提交认证</view>
				</view>
				<view v-if="isOk"
					class="info_box">
					<view class="color_000 fontS26 bold mar_top30">真实姓名：{{userInfo.nickname}}</view>
					<view class="color_000 fontS26 bold mar_top30">身份证号：{{userInfo.nickname}}</view>
					<view class="color_000 fontS26 bold mar_top30">银行卡号：{{userInfo.nickname}}</view>
				</view>
			</view>
		</view>
		<z-model :show="isSure"
			@close="isSure = false"
			content="确认提交吗？"
			@click="sureSubmit"></z-model>
	</view>
</template>
<script>
	import {
		mapGetters
	} from "vuex";
	export default {
		data() {
			return {
				ossurl: "https://zhongshuspace.oss-cn-beijing.aliyuncs.com",
				url: "https://app.zskj.art",
				okImgUrl: "https://app.zskj.art/static/image/common/ok.png",
				yesImgUrl: "https://app.zskj.art/static/image/common/yes.png",
				formData: {
					user_name: '',
					id_num: ''
				},
				isOk: false,
				isSure: false,
				card: '',
				loading: true,
				backgroundImage: 'https://app.zskj.art/static/image/common/my_back1.png',
			}
		},
		computed: mapGetters(['isLogin', 'chatUrl', 'userInfo', 'uid']),
		onLoad() {
			console.log("userInfo:", this.userInfo);
			this.loading = false
			this.isOk = false
		},
		methods: {
			
			submit() {
				console.log("pageObj = this-- formData", this.formData);
				if (!this.formData.user_name) {
					uni.showToast({
						title: '请输入真实姓名',
						duration: 2000
					});
				}
				if (!this.formData.id_num) {
					uni.showToast({
						title: '请输入身份证号',
						duration: 2000
					});
				} else {
					this.isSure = true;
					console.log("this.isSure :", this.isSure);
				}
			},
			sureSubmit() {
				this.isSure = false
				this.$loading('实名认证中...')
				this.$base.tips('实名认证成功！', 'success', false)
				this.isOk = true
				// RealAuthentication(this.formData).then(res => {
				// 	this.$base.tips('实名认证成功！', 'success', false)
				// 	this.getUserInfo('', res => {
				// 		this.isOk = true
				// 	})
				// })
			}
		}
	}
</script>
<style lang="scss"
	scoped>
	page {
		font-family: PingFang SC;
	}

	.height_85 {
		height: 85rpx;
	}

	.submit {
		width: 100%;
		height: 80rpx;
		opacity: 1;
	}

	.my_info_box {
		width: 100%;
		height: 560rpx;
		color: #fff;

		.info_box {
			padding: 0 79rpx 0 71rpx;
		}
	}

	.hander_box {
		width: 100%;
		height: 160rpx;
		background-size: 100% 100%;
		background-repeat: no-repeat;
	}
</style>
