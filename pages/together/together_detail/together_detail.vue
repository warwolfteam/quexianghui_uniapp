<template>
	<view>
		{{info.activity}}
	</view>
</template>
<script>
	export default {
		data() {
			return {
				info: {
					activity: null,
					activityH5: null,
					cartNum: null,
					ficti: null,
					flatPattern: null,
					id: null,
					image: null,
					otPrice: null,
					price: null,
					sales: null,
					stock: null,
					storeName: null,
					unitName: null,
				},
			};
		},
		onLoad(option) {
			var that = this;
			console.log("输出传入参数：", option);
			const item = JSON.parse(decodeURIComponent(option.item));
			console.log("格式化输出传入参数：", item);
			that.info = item;
		},
	}
</script>
<style lang="scss">
</style>
