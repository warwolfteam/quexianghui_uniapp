<template>
	<view>
		<view v-if="isShow" class="">
			<u-popup :show="isShow" @close="isShow = false" @open="isShow = true" mode="center"
				border-radius="30" :closeOnClickOverlay="closeOnClickOverlay"> 
				<view class="bg_color_fff paddding border_radius_30" style="width: 600rpx;">
					<view class="color_000 fontS26 bold mar_top40 text-center">温馨提示</view>
					<view class="fontS32 bold mar_top70 text-center" :style="{color:color}">{{content}}</view>
					<view @click="sureClick" class="bgColorjb border_radius_20 sure_btn">{{confirmText}}</view>
				</view>
				<view class=" bold fontS32 mar_top30" style="color: #000;text-align: center;">
					<view class="flex align-center justify-center" @click="close">
						<u-icon :name="`/static/img/close_p.png`" size="30"></u-icon>
					</view>
				</view>
			</u-popup>
		</view>
	</view>
</template>

<script>
	export default {
		name: "z-model",
		props:{  
			confirmText:{
				type:String,
				default:'确定'
			},
			closeOnClickOverlay:{
				type:Boolean,
				default:false
			},
			content:{
				type:String,
				default:''
			},
			color:{
				type:String,
				default:'#000'
			},
			pageObj:{
				type:Object,
				default:() => {
					return {}
				}
			},
			show:{
				type:Boolean,
				default:false
			}
		},
		watch:{
			show(value){
				this.isShow = value
			}
		},
		data() {
			return {
				isShow:this.show
			};
		},
		methods:{
			sureClick(){
				this.$emit('click')
			},
			close(){
				// this.show = false
				 this.$emit('close', false); 
			}
		}
	}
</script>

<style scoped lang="scss">
	.sure_btn {
		width: 320rpx;
		height: 80rpx;
		opacity: 1;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		line-height: 48rpx;
		color: #FFFFFF;
		opacity: 1;
		text-align: center;
		line-height: 80rpx;
		margin: 90rpx auto 40rpx;
	}
</style>
