<template>
	<view>
		<view class="cropper-wrapper">
			<canvas
				class="cropper"
				disable-scroll="true"
				@touchstart="touchStart($event)"
				@touchmove="touchMove($event)"
				@touchend="touchEnd($event)"
				:style="{width: cropperOption.width + 'px',height: cropperOption.height + 'px', backgroundColor: 'rgba(0, 0, 0, 0.8)'}"
				:canvas-id="cropperOption.id">
			</canvas>
			<view class="cropper-buttons">
				<view class="upload" @click="uploadTap">重新选择</view>
				<view class="getCropperImage" @click="getCropperImage">确定</view>
			</view>
		 </view>
	</view>
</template>

<script>
	import weCropper from '../../helper/cropper.js'
	const device = uni.getSystemInfoSync()
	const width = device.windowWidth
	const height = device.windowHeight -50
	
	export default {
		data() {
			return {
				cropperOption: {
					id: 'cropper',
					width,
					height,
					scale: 2.5,
					zoom: 8,
					cut: {
						x: (width - 300) / 2,
						y: (height - 300) / 2,
						width: 300,
						height: 300
					}
				},
				wecropper: null,
				name: '',
				i: 0
			}
		},
		methods: {
			getCropperImage: function() {
				this.wecropper.getCropperImage((avatar) => {
					if (avatar) {
						uni.redirectTo({
							url: `../select/select?avatar=${avatar}&i=${this.i}`
						})
					} else {
						console.log('获取图片失败')
					}
				})
			},
			uploadTap: function() {
				uni.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'],
					sourceType: ['album', 'camera'],
					success: (res) => {
						this.wecropper.pushOrign(res.tempFilePaths[0])
					}
				})
			},
			touchStart: function(e) {
				this.wecropper.touchStart(e)
			},
			touchMove: function(e) {
				this.wecropper.touchMove(e)
			},
			touchEnd: function(e) {
				this.wecropper.touchEnd(e)
			}
		},
		onLoad: function(option) {
			this.name = option.name
			this.i = option.i
			const {cropperOption} = this
			const {src} = option
			if(src) {
				Object.assign(cropperOption, {src})
				this.wecropper = new weCropper(cropperOption)
				.on('ready', (ctx) => {
					
				})
				.on('beforeImageLoad', (ctx) => {
					uni.showToast({
						title: '上传中',
						icon:'loading',
						duration: 20000
					})
				})
				.on('imageLoad', (ctx) => {
					uni.hideToast()
				})
			}
		}
	}
</script>

<style>
.cropper-wrapper{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    justify-content: center;
    height: 100%;
    background-color: #e5e5e5;
}

.cropper-buttons{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    justify-content: center;
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 50px;
    line-height: 50px;
}

.cropper-buttons .upload, .cropper-buttons .getCropperImage{
    width: 50%;
    text-align: center;
}
.cropper{
	position: absolute;
    top: 0;
    left: 0;
}

.cropper-buttons{
    background-color: rgba(0, 0, 0, 0.95);
    color: #04b00f;
}
</style>
