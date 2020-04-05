<template>
	<view>
		<view class="">
			<canvas
				class="cropper"
				disable-scroll="true"
				@touchstart="touchStart($event)"
				@touchmove="touchMove($event)"
				@touchend="touchEnd($event)"
				:style="{width: cropperOption.width + 'px',height: cropperOption.height + 'px', backgroundColor: 'rgba(0, 0, 0, 0.8)'}"
				:canvas-id="cropperOption.id">
			</canvas>
		</view>
		<view class="cropper-wrapper">
			<view class="cropper-buttons">
				<view class="upload" @click="uploadTap">重新选择</view>
				<view class="getCropperImage" @click="getCropperImage">确定</view>
			</view>
		 </view>
	</view>
</template>

<script>
	import weCropper from '../../helper/cropper.js'
	
	export default {
		data() {
			return {
				device: null,
				cropperOption: {
					id: 'cropper',
					width: 0,
					height: 0,
					scale: 2.5,
					zoom: 8,
					cut: {
						x: (this.width - 300) / 2,
						y: (this.height - 300) / 2,
						width: 300,
						height: 300
					}
				}
			}
		},
		methods: {
			getCropperImage: function() {
				this.wecropper.getCropperImage((avatar) => {
					if (avatar) {
						uni.redirectTo({
							url: `../select/select?avatar=${avatar}`
						})
					} else {
						console.log('获取图片失败')
					}
				})
			},
			uploadTap: function() {
				console.log(this.wecropper)
				uni.chooseImage({
					count: 1,
					success: (res) => {
						this.wecropper.pushOrign(res.tempFilePaths[0])
					}
				})
			},
			touchStart: function(e) {
				console.log(e)
				this.wecropper.touchStart(e)
			},
			touchMove: function(e) {
				this.wecropper.touchStart(e)
			},
			touchEnd: function(e) {
				console.log(e)
				this.wecropper.touchStart(e)
			}
		},
		onLoad: function(option) {
			this.device = uni.getSystemInfoSync()
			this.cropperOption.width = this.device.windowWidth
			this.cropperOption.height = this.device.windowHeight - 50
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
    top: 0;
    left: 0;
}

.cropper-buttons{
    background-color: rgba(0, 0, 0, 0.95);
    color: #04b00f;
}
</style>
