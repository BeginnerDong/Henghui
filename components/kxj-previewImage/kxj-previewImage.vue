<template>
	<view class="previewImage" :style="{ opacity: opacity}" v-if="show" @tap="close" @touchmove.stop.prevent >
		<view class="swiper">
			<view v-for="(img, i) in imgs" :key="i" >
				<movable-area class="marea" scale-area>
					<movable-view class="mview" direction="all" scale="true" @scale="onScale" scale-min="0.5" scale-max="4" :scale-value="scale" >
						<image class="image" :src="img" :data-index="i" :data-src="img" mode="widthFix" @touchmove="handletouchmove" @touchstart="handletouchstart" @touchend="handletouchend" />
					</movable-view>
				</movable-area>
			</view>
		</view>
		<view class="page" v-if="imgs.length > 0">
			<text class="text">{{ 1 }} / {{ imgs.length }}</text>
		</view>
		<view class="desc" v-if="descs.length > 0 && descs.length == imgs.length&&descs[index].length>0">{{ descs[index] }}</view>
	</view>
</template>

<script>
export default {
	name: 'ksj-previewImage', //插件名称
	props: {
		imgs: {//图片列表
			type: Array,
			required: true,
			default: () => {
				return [];
			}
		},
		descs: {//描述列表
			type: Array,
			required: false,
			default: () => {
				return [];
			}
		},
		//透明度,0到1之间。
		opacity: {
			type: Number,
			default: 0.8
		},
		height: {
			type: Number,
			required: false,
		}
	},
	data() {
		return {
			show: false, //显示状态
			index: 0, //当前页
			time:0,//定时器
			interval: 1000, //长按事件
			scale: 1, //缩放比例
			old: {
				scale: 1 //缩放比例
			},
			x:0,
			y:0
		};
	},
	methods: {
		onScale(e) {
			console.log(e)
		
			this.old.scale = e.detail.scale
		},
		//接触开始
		handletouchstart(e) {
			var tchs = e.touches.length;
			if (tchs != 1) {
				return false;
			}
			this.time = setTimeout(() => {
				this.onLongPress(e);
			}, this.interval);
			return false;
		},
		//清除定时器
		handletouchend() {
			clearTimeout(this.time);
			if (this.time != 0) {
				//处理点击时间
			}
			return false;
		},
		//清除定时器
		handletouchmove() {
			clearTimeout(this.time);
			this.time = 0;
		},
		// 处理长按事件
		onLongPress(e) {
			var src = e.currentTarget.dataset.src;
			var index = e.currentTarget.dataset.index;
			var data={src:src,index:index}
			this.$emit('longPress', data);
		},
		//图片改变
		change(e) {
			this.scale = 1;
			this.index = e.target.current;
		},
		//打开
		open(e) {
			if (e===null||e==="") {
				return;
			}
			
			if(!isNaN(e)){
				this.index = e;
			}else{
				this.index = this.imgs.indexOf(e);
			}
			console.log(this.index)
			this.show = true;
		},
		//关闭
		close(e) {
			this.show = false;
		}
	}
};
</script>

<!--使用scss,只在本组件生效-->
<style lang="scss" scoped>
.previewImage {
	z-index: 999;
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: #000000;
	user-select: none;
	.swiper {
		width: 100%;
		height: 100%;
		.marea {
			height: 100%;
			width: 100%;
			position: fixed;
			overflow-y: auto;
			
			.mview {
				display: flex;
				align-items: center;
				justify-content: center;
			
				width: 100%;
				height: 100%;
				.image {
					width: 100%;
				}
			}
		}
	}

	.page {
		position: absolute;
		width: 100%;
		bottom: 20rpx;
		text-align: center;
		.text {
			color: #fff;
			font-size: 26rpx;
			background-color: rgba(0, 0, 0, 0.5);
			padding: 3rpx 16rpx;
			border-radius: 20rpx;
		}
	}
	.desc {
		position: absolute;
		top: 0;
		width: 100%;
		padding: 5rpx 10rpx;
		text-align: center;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		background-color: rgba(0, 0, 0, 0.5);
		color: #fff;
		font-size: 28rpx;
		letter-spacing: 3rpx;
	}
}
</style>
