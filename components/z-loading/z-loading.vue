<template>
	<view>
		<view class="" :style="{width:width,height:height}" class="flex align-center justify-center">
			<u-loading :text="text" textSize="18"></u-loading>
		</view>
	</view>
</template>

<script>
	export default {
		name:"z-loading",
		props:{
			width:{
				type:String,
				default:'100%'
			},
			height:{
				type:String,
				default:'90vh'
			},
			textSize:{
				type:String,
				default:'18rpx'
			},
			text:{
				type:String,
				default:'加载中'
			}
		},
		data() {
			return {
				
			};
		}
	}
</script>

<style>

</style>