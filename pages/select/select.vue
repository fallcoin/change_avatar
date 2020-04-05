<template>
	<view>
		<view class="user_img_content">
			<image :src="uploadImg" class="user_avater"></image>
			<image :src="borderImg" class="user_border"></image>
		</view>
		<button type="default" @click="chooseImage">选择图像</button>
		<button type="default" @click="generateImage">生成图像</button>
		
		<view class="hidden-canvas">
		  <canvas :canvas-id="canvasId" :style="{width: canvas.width + 'px', height: canvas.height + 'px', position: 'absolute', left: '-99999999px'}"></canvas>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				uploadImg: '',
				borderImg: '../../static/image/avatar_border/1.png',
				ctx: null,
				canvasId: 'canvas',
				canvas: {
					width: 0,
					height: 0,
					background: 'rgb(255,255,255)'
				}
			}
		},
		methods: {
			chooseImage: function() {
				uni.chooseImage({
					count: 1,
					success: (res) => {
						this.uploadImg = res.tempFilePaths[0];
						uni.navigateTo({
							url: `../upload/upload?src=${this.uploadImg}`
						})
					},
					fail: () => {
						uni.showToast({
							title: '获取图片失败'
						})
					}
				})
			},
			generateImage: function() {
				this.canvas.width = 840
				this.canvas.height = 840
				this.ctx = uni.createCanvasContext(this.canvasId)
				this.ctx.drawImage(this.uploadImg, 127, 120)
				this.ctx.restore()
				this.ctx.save()
				this.ctx.beginPath()
				this.ctx.drawImage(this.borderImg, 0, 0, this.canvas.width, this.canvas.height)
				this.ctx.restore()
				this.ctx.draw(true, setTimeout(() => {
					uni.canvasToTempFilePath({
						width: 840,
						height: 840,
						destWidth: 840,
						destHeight: 840,
						canvasId: this.canvasId,
						success: res => {
							uni.saveImageToPhotosAlbum({
								filePath: res.tempFilePath,
								success: (res) => {
									uni.showToast({
										title: '保存成功'
									})
								},
								fail: (err) => {
									if(err.errMsg === "saveImageToPhotosAlbum:fail auth deny") {
										uni.openSetting({
											success: (settingdata) => {
												if(settingdata.authSetting['scope.writePhotosAlbum']) {
													uni.showToast({
														title: '请再次保存'
													})
												} else {
													uni.showToast({
														title: '获取权限失败，可能导致保存图片无法正常使用'
													})
												}
											}
										})
									}
								}
							})
						}
					}, this)
				}, 100))
			}
		}
	}
</script>

<style>
	.user_img_content {
		position: relative;
		width: 200rpx;
		height: 200rpx;
	}
	.user_avater, .user_border {
		width: 200rpx;
		height: 200rpx;
		border-radius: 50%;
	}
	.user_border {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
	}
</style>
