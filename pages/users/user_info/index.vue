<template>
	<view>
		<form @submit="formSubmit"
			report-submit='true'>
			<view class='personal-data pad30'>
				<view class='list borRadius14'>
					<view class="item acea-row row-between-wrapper">
						<view>头像</view>
						<view class="pictrue"
							@click.stop='uploadpic'>
							<image :src='newAvatar ? newAvatar : userInfo.avatar'></image>
							<image src='../../../static/images/alter.png'
								class="alter"></image>
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>ID号</view>
						<view class='input acea-row row-between-wrapper'>
							<input type='text'
								:value='uid'
								disabled='true'
								class='id'></input>
							<text class='iconfont icon-suozi'></text>
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>昵称</view>
						<view class='input'><input type='text'
								name='nickname'
								:value='userInfo.nickname'></input>
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>性别</view>
						<view class="input"
							@click="setSex"
							v-if="!userInfo.sex"> 点击选择性别 </view>
						<view class="input"
							@click="setSex"
							v-else> {{userInfo.sex}}
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>年龄</view>
						<view class="input"
							@click="setAge"
							v-if="!userInfo.age"> 点击选择年龄 </view>
						<view class="input"
							@click="setAge"
							v-else> {{userInfo.age}}
						</view>
					</view>
					<view class='item acea-row row-between-wrapper relative'>
						<view>地区</view>
						<view class="address">
							<picker mode="multiSelector"
								@change="bindRegionChange"
								@columnchange="bindMultiPickerColumnChange"
								:value="valueRegion"
								:range="multiArray">
								<view class='acea-row'>
									<view class="input">{{region[0]}}，{{region[1]}}，{{region[2]}}</view>
								</view>
							</picker>
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>微信号</view>
						<view class='input'><input type='text'
								placeholder="请输入微信号"
								name='wechatId'
								:value='userInfo.wechatId'></input>
						</view>
					</view>
					<view class='item acea-row row-between-wrapper'>
						<view>手机号码</view>
						<navigator url="/pages/users/app_login/index"
							hover-class="none"
							class="input"
							v-if="!userInfo.phone"> 点击绑定手机号<text class="iconfont icon-xiangyou"></text>
						</navigator>
						<navigator url="/pages/users/user_phone/index"
							hover-class="none"
							class="input"
							v-else>
							<view class='input acea-row row-between-wrapper'>
								<input type='text'
									disabled='true'
									name='phone'
									:value='userInfo.phone'
									class='id'></input>
								<text class='iconfont icon-xiangyou'></text>
							</view>
						</navigator>
					</view>
					<!-- #ifdef MP -->
					<view class='item acea-row row-between-wrapper'>
						<view>权限设置</view>
						<view class="input"
							@click="Setting"> 点击管理<text class="iconfont icon-xiangyou"></text>
						</view>
					</view>
					<!-- #endif -->
					<view class="item acea-row row-between-wrapper"
						v-if="userInfo.phone">
						<view>密码</view>
						<navigator url="/pages/users/user_pwd_edit/index"
							hover-class="none"
							class="input"> 点击修改密码<text class="iconfont icon-xiangyou"></text>
						</navigator>
					</view>
				</view>
				<button class='modifyBnt bg-color'
					formType="submit">保存修改</button>
				<!-- #ifdef H5 -->
				<view class="logOut cart-color acea-row row-center-wrapper"
					@click="outLogin"
					v-if="!this.$wechat.isWeixin()">退出登录</view>
				<!-- #endif -->
			</view>
		</form>
		<u-select v-model="showSetAge"
			mode="single-column"
			@confirm="confirmSetAge"
			:list="ageList"></u-select>
		<u-select v-model="showSetSex"
			mode="single-column"
			@confirm="confirmSetSex"
			:list="sexList"></u-select>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
	</view>
</template>
<script>
	import {
		userEdit,
		getLogout
	} from '@/api/user.js';
	import {
		switchH5Login
	} from '@/api/api.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		getCity
	} from '@/api/api.js';
	import {
		mapGetters
	} from "vuex";
	import dayjs from "@/plugin/dayjs/dayjs.min.js";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
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
			};
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
		onLoad() {
			if (!this.isLogin) {
				toLogin();
			} else {
				this.setAgeList();
				this.getCityList();
				console.log("userInfo", this.userInfo);
			}
		},
		methods: {
			setAgeList() {
				console.log("初始ageList：", this.ageList);
				var arr = [];
				for (var i = 18; i < 200; i++) {
					arr.push({
						label: i + "岁",
						value: i
					})
				}
				console.log("造arr：", arr);
				this.ageList = arr;
				console.log("ageList生成完毕：", this.ageList);
			},
			// 点击选择年龄
			setAge() {
				console.log("点击选择年龄");
				this.showSetAge = true;
			},
			confirmSetAge(e) {
				console.log("点击选择年龄", e);
				console.log("点击选择年龄", e[0].label);
				this.userInfo.age = e[0].label
			},
			// 设置性别
			setSex() {
				console.log("点击选择性别");
				this.showSetSex = true;
			},
			confirmSetSex(e) {
				console.log("点击选择性别", e);
				console.log("点击选择性别", e[0].label);
				this.userInfo.sex = e[0].label
			},
			// 点击地区
			// #ifdef APP-PLUS
			// 获取选择的地区
			handleGetRegion(region) {
				this.region = region
			},
			// #endif
			// 获取地址数据
			getCityList: function() {
				let that = this;
				getCity().then(res => {
					console.log("getCity:", res);
					this.district = res.data;
					console.log("this.district:", this.district);
					let oneDay = 24 * 3600 * 1000;
					// this.$Cache.set('cityList', JSON.stringify(res.data)); //设置不过期时间的方法 
					this.$Cache.setItem({
						name: 'cityList',
						value: res.data,
						expires: oneDay * 7
					}); //设置七天过期时间
					that.initialize();
				})
			},
			initialize: function() {
				let that = this,
					province = [],
					city = [],
					area = [];
				if (that.district.length) {
					let cityChildren = that.district[0].child || [];
					let areaChildren = cityChildren.length ? (cityChildren[0].child || []) : [];
					that.district.forEach(function(item) {
						province.push(item.name);
					});
					cityChildren.forEach(function(item) {
						city.push(item.name);
					});
					areaChildren.forEach(function(item) {
						area.push(item.name);
					});
					this.multiArray = [province, city, area]
					console.log("this.multiArray:", this.multiArray);
				}
			},
			bindRegionChange: function(e) {
				console.log("bindRegionChange:", e);
				let multiIndex = this.multiIndex,
					province = this.district[multiIndex[0]] || {
						child: []
					},
					city = province.child[multiIndex[1]] || {
						cityId: 0
					},
					multiArray = this.multiArray,
					value = e.detail.value;
				this.region = [multiArray[0][value[0]], multiArray[1][value[1]], multiArray[2][value[2]]]
				this.cityId = city.cityId
				this.valueRegion = [0, 0, 0]
				this.initialize();
			},
			bindMultiPickerColumnChange: function(e) {
				console.log("bindMultiPickerColumnChange:", e);
				let that = this,
					column = e.detail.column,
					value = e.detail.value,
					currentCity = this.district[value] || {
						child: []
					},
					multiArray = that.multiArray,
					multiIndex = that.multiIndex;
				multiIndex[column] = value;
				switch (column) {
					case 0:
						let areaList = currentCity.child[0] || {
							child: []
						};
						multiArray[1] = currentCity.child.map((item) => {
							return item.name;
						});
						multiArray[2] = areaList.child.map((item) => {
							return item.name;
						});
						break;
					case 1:
						let cityList = that.district[multiIndex[0]].child[multiIndex[1]].child || [];
						multiArray[2] = cityList.map((item) => {
							return item.name;
						});
						break;
					case 2:
						break;
				}
				// #ifdef MP || APP-PLUS
				this.$set(this.multiArray, 0, multiArray[0]);
				this.$set(this.multiArray, 1, multiArray[1]);
				this.$set(this.multiArray, 2, multiArray[2]);
				// #endif
				// #ifdef H5
				this.multiArray = multiArray;
				// #endif
				this.multiIndex = multiIndex
				// this.setData({ multiArray: multiArray, multiIndex: multiIndex});
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			/**
			 * 小程序设置
			 */
			Setting: function() {
				uni.openSetting({
					success: function(res) {
						console.log(res.authSetting)
					}
				});
			},
			/**
			 * 退出登录
			 * 
			 */
			outLogin: function() {
				let that = this;
				if (that.loginType == 'h5') {
					uni.showModal({
						title: '提示',
						content: '确认退出登录?',
						success: function(res) {
							if (res.confirm) {
								getLogout().then(res => {
									that.$store.commit("LOGOUT");
									uni.reLaunch({
										url: '/pages/index/index'
									});
								}).catch(err => {
									console.log(err);
								});
							} else if (res.cancel) {
								console.log('用户点击取消');
							}
						}
					});
				}
			},
			/**
			 * 上传文件
			 * 
			 */
			uploadpic: function() {
				let that = this;
				console.log("进入上传文件方法");
				that.$util.uploadImageOne({
					url: 'user/upload/image',
					name: 'multipart',
					model: "maintain",
					pid: 0
				}, function(res) {
					that.newAvatar = res.data.url;
				});
			},
			/**
			 * 提交修改
			 */
			formSubmit: function(e) {
				console.log("提交修改", e);
				let that = this,
					value = e.detail.value
				if (!value.nickname) return that.$util.Tips({
					title: '用户姓名不能为空'
				});
				value.avatar = that.newAvatar ? that.newAvatar : that.userInfo.avatar;
				value.sex = that.userInfo.sex;
				value.age = that.userInfo.age;
				value.region = that.region;
				userEdit(value).then(res => {
					that.$store.commit("changInfo", {
						amount1: 'avatar',
						amount2: that.newAvatar
					});
					return that.$util.Tips({
						title: '更换头像已成功',
						icon: 'success'
					}, {
						tab: 3,
						url: 1
					});
				}).catch(msg => {
					return that.$util.Tips({
						title: msg || '保存失败，您并没有修改'
					}, {
						tab: 3,
						url: 1
					});
				});
			}
		}
	}
</script>
<style scoped
	lang="scss">
	.personal-data .wrapper {
		margin: 10rpx 0;
		background-color: #fff;
		padding: 36rpx 30rpx 13rpx 30rpx;
	}

	.personal-data .wrapper .title {
		margin-bottom: 30rpx;
		font-size: 32rpx;
		color: #282828;
	}

	.personal-data .wrapper .wrapList .item {
		width: 690rpx;
		height: 160rpx;
		background-color: #f8f8f8;
		border-radius: 20rpx;
		margin-bottom: 22rpx;
		padding: 0 30rpx;
		position: relative;
		border: 2rpx solid #f8f8f8;
		box-sizing: border-box;
	}

	.personal-data .wrapper .wrapList .item.on {
		border-color: $theme-color;
		border-radius: 20rpx;
		background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAArIAAACgCAYAAADw+I85AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyhpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTQ1IDc5LjE2MzQ5OSwgMjAxOC8wOC8xMy0xNjo0MDoyMiAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTkgKE1hY2ludG9zaCkiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6M0QzNkY3NzlCNzJCMTFFOTgyNEU4QzhGQTRFRUY2REQiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6M0QzNkY3N0FCNzJCMTFFOTgyNEU4QzhGQTRFRUY2REQiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDozRDM2Rjc3N0I3MkIxMUU5ODI0RThDOEZBNEVFRjZERCIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDozRDM2Rjc3OEI3MkIxMUU5ODI0RThDOEZBNEVFRjZERCIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Pn3rJMAAAArUSURBVHja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");
		background-size: 100% 100%;
		background-color: #fff9f9;
		background-repeat: no-repeat;
	}

	.personal-data .wrapper .wrapList .item .picTxt {
		width: 445rpx;
	}

	.personal-data .wrapper .wrapList .item .picTxt .pictrue {
		width: 96rpx;
		height: 96rpx;
		position: relative;
	}

	.personal-data .wrapper .wrapList .item .picTxt .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
	}

	.personal-data .wrapper .wrapList .item .picTxt .pictrue .alter {
		width: 30rpx;
		height: 30rpx;
		border-radius: 50%;
		position: absolute;
		bottom: 0;
		right: 0;
	}

	.personal-data .wrapper .wrapList .item .picTxt .text {
		width: 325rpx;
	}

	.personal-data .wrapper .wrapList .item .picTxt .text .name {
		width: 100%;
		font-size: 30rpx;
		color: #282828;
	}

	.personal-data .wrapper .wrapList .item .picTxt .text .phone {
		font-size: 24rpx;
		color: #999;
		margin-top: 10rpx;
	}

	.personal-data .wrapper .wrapList .item .bnt {
		font-size: 24rpx;
		background-color: #fff;
		border-radius: 27rpx;
		width: 140rpx;
		height: 54rpx;
		border: 2rpx solid $theme-color;
	}

	.personal-data .wrapper .wrapList .item .currentBnt {
		position: absolute;
		right: 0;
		top: 0;
		font-size: 26rpx;
		background-color: rgba(233, 51, 35, 0.1);
		width: 140rpx;
		height: 48rpx;
		border-radius: 0 20rpx 0 20rpx;
	}

	.personal-data .list {
		margin-top: 30rpx;
		background-color: #fff;
	}

	.personal-data .list .item {
		border-bottom: 1rpx solid #f2f2f2;
		padding: 24rpx;
		font-size: 32rpx;
		color: #282828;
	}

	.personal-data .list .item .phone {
		width: 160rpx;
		height: 56rpx;
		font-size: 24rpx;
		color: #fff;
		line-height: 56rpx;
		border-radius: 32rpx
	}

	.personal-data .list .item .pictrue {
		width: 88rpx;
		height: 88rpx;
		position: relative;
	}

	.personal-data .list .item .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
	}

	.personal-data .list .item .pictrue .alter {
		width: 30rpx;
		height: 30rpx;
		border-radius: 50%;
		position: absolute;
		bottom: 0;
		right: 0;
	}

	.personal-data .list .item .input {
		width: 415rpx;
		text-align: right;
		color: #868686;
	}

	.personal-data .list .item .input .id {
		width: 365rpx;
	}

	.personal-data .list .item .input .iconfont {
		font-size: 35rpx;
	}

	.personal-data .modifyBnt {
		font-size: 32rpx;
		color: #fff;
		width: 690rpx;
		height: 90rpx;
		border-radius: 50rpx;
		text-align: center;
		line-height: 90rpx;
		margin: 76rpx auto 0 auto;
	}

	.personal-data .logOut {
		font-size: 32rpx;
		text-align: center;
		width: 690rpx;
		height: 90rpx;
		border-radius: 45rpx;
		margin: 30rpx auto 0 auto;
	}
</style>
