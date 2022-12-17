<template>
	<view>
		<input v-if="type == 'text'"
			v-model="valueModel"
			:disabled="disabled"
			class="input"
			:placeholder="placeholder"
			:maxlength="maxlength"
			placeholder-class="placeholderclass"
			:style="{color:color,backgroundColor:bgColor,height:height,borderRadius:radius,borderColor:borderColor,textAlign:textAlign,border:border,marginTop:marginTop,width:width,padding:padding,fontSize:fontSize}"
			type="text"
			@click="sureClick"
			@blur="blur"></input>
		<input v-if="type == 'number'"
			v-model="valueModel"
			:disabled="disabled"
			class="input"
			:placeholder="placeholder"
			:maxlength="maxlength"
			placeholder-class="placeholderclass"
			:style="{color:color,backgroundColor:bgColor,height:height,borderRadius:radius,borderColor:borderColor,textAlign:textAlign,border:border,marginTop:marginTop,width:width,padding:padding}"
			type="number"
			@click="sureClick"></input>
		<input v-if="type == 'password'"
			v-model="valueModel"
			:disabled="disabled"
			class="input"
			:placeholder="placeholder"
			:maxlength="maxlength"
			placeholder-class="placeholderclass"
			:style="{color:color,backgroundColor:bgColor,height:height,borderRadius:radius,borderColor:borderColor,textAlign:textAlign,border:border,marginTop:marginTop,width:width,padding:padding}"
			type="password"></input>
	</view>
</template>
<script>
	export default {
		name: "z-input",
		props: {
			disabled: {
				type: Boolean,
				default: false
			},
			color: {
				type: String,
				default: "#000"
			},
			fontSize: {
				type: String,
				default: "28rpx"
			},
			maxlength: {
				type: [Number, String],
				default: '-1'
			},
			type: {
				type: String,
				default: 'text'
			},
			value: {
				type: [Array, String, Number, Date, Boolean],
				default: ''
			},
			placeholder: {
				type: String,
				default: '请输入内容'
			},
			bgColor: {
				type: String,
				default: '#fff'
			},
			width: {
				type: String,
				default: '100%'
			},
			height: {
				type: String,
				default: '80rpx'
			},
			radius: {
				type: String,
				default: '20rpx'
			},
			borderColor: {
				type: String,
				default: '#E0E0E0'
			},
			textAlign: {
				type: String,
				default: 'center'
			},
			border: {
				type: String,
				default: '1px solid #333333'
			},
			marginTop: {
				type: String,
				default: '20rpx'
			},
			padding: {
				type: String,
				default: '10rpx 20rpx'
			},
		},
		model: {
			//需要定义哪一个props可以用v-model绑定属性
			prop: 'value',
			//声明一个事件，调用之后，就会改变父级容器的值
			event: 'handlerValueChange'
		},
		computed: {
			valueModel: {
				get: function() {
					//实际使用的是value值
					return this.value;
				},
				set: function(val) {
					//侦听到setter()事件，将值传递回父级组件
					this.$emit('handlerValueChange', val);
				}
			}
		},
		data() {
			return {};
		},
		methods: {
			sureClick() {
				this.$emit('click')
			},
			blur(event) {
				let value = event.detail.value
				console.log("event:", event);
				console.log("value:", value);
				this.$emit('blur', value)
			}
		}
	}
</script>
<style scoped
	lang="scss">
	.input {
		// background-color: #17182A;
		// height: 80rpx;
		// border-radius: 40rpx;
		// border: 1px solid #41435F;
		// border-color: #41435F;
		// text-align: center;
		// color: #FFFFFF;
		padding: 10rpx 20rpx;
		// margin-top: 55rpx;
	}

	.placeholderclass {
		color: #BDBDBD;
		font-size: 32rpx;
	}
</style>
