<template>
	<view class="paddingB30">
		<view v-for="(item,index) in list"
			:key="index"
			class="response paddding bg_color_fff border_radius_30 mar_top30">
			<view class="flex align-center justify-between">
				<view class="flex align-center">
					<text class="fontS32 bold mar_right20"
						:class="{color_828282:item.read_status!=0}">{{item.title}}</text>
					<view class="no_red mar_bottom20"
						v-if="item.read_status==0"></view>
				</view>
				<view class="fontS22"
					:class="{color_828282:item.read_status!=0}">{{item.createtime}}</view>
			</view>
			<view class="fontS26 mar_top25"
				:class="{color_828282:item.read_status!=0}">{{item.content}}</view>
		</view>
	</view>
</template>
<script>
	export default {
		name: "z-messageList",
		props: {
			list: {
				type: Array,
				default () {
					return []
				}
			}
		},
		data() {
			return {};
		}
	}
</script>
<style lang="scss"
	scoped>
	page {
		font-family: PingFang SC;
	}

	.no_red {
		width: 14rpx;
		height: 14rpx;
		background: red;
		border-radius: 100%;
		opacity: 1;
	}
</style>
