<template>
	<view 
		class="container" 
		:style="{height: height + 'px'}" 
		@touchstart = "handletouchtart($event)" 
		@touchmove="handletouchmove($event)" 
		@touchend="handletouchend($event)"
	>
		<!-- 当前的头像框，固定在页面中间 -->
		<view class="border_main">
			<image :src="uploadImg" class="user_avatar"></image>
			<image :src="currentBorderImg.showurl" mode="" class="user_avatar_border"></image>
		</view>
		<!-- 进行上下滑动的内容 -->
		<view class="content" :animation="slide" :style="{height: 439 + 'px', transform: back}">
			<!-- 页面上半部分 -->
			<view class="part_top" :style="{height: height + 'px'}">
				<view class="tip">
					<view><text class="boldText">左右</text>滑动选择心仪的框</view>
					<view><text class="boldText">向上</text>滑动进入头像制作</view>
					<view>共有<text class="boldText">6</text>款头像框可供选择</view>
				</view>
				<view class="boderlist">
					<view class="border border_left">
						<image :src="borderImgs[(i) % borderImgs.length].showurl" mode=""></image>
					</view>
					<view class="border border_left2">
						<image :src="borderImgs[(i + 1) % borderImgs.length].showurl" mode=""></image>
					</view>
					<view class="border border_right2">
						<image :src="borderImgs[(i + 3) % borderImgs.length].showurl" mode=""></image>
					</view>
					<view class="border border_right">
						<image :src="borderImgs[(i + 4) % borderImgs.length].showurl" mode=""></image>
					</view>
				</view>
			</view>
			<!-- 页面下半部分 -->
			<view class="part_bottom" :style="{height: height/2 + 'px'}">
				<view class="btn">
					<view @click="chooseImage" class="btn-choose">选择图像</view>
					<view @click="generateImage" class="btn-generate">生成图像</view>
				</view>
				<!-- 隐藏的画布区域 -->
				<view class="hidden-canvas">
				  <canvas :canvas-id="canvasId" :style="{width: canvas.width + 'px', height: canvas.height + 'px', position: 'absolute', left: '-99999999px'}"></canvas>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				uploadImg: '',	// 用户上传的图片
				currentBorderImg: '',	// 当前头像框
				i: 0,
				borderImgs: [
					{
						url: '../../static/image/avatar_border/1.png',
						showurl: '../../static/image/avatar_border/show_1.png'
					},
					{
						url: '../../static/image/avatar_border/2.png',
						showurl: '../../static/image/avatar_border/show_2.png'
					},
					{
						url: '../../static/image/avatar_border/3.png',
						showurl: '../../static/image/avatar_border/show_3.png'
					},
					{
						url: '../../static/image/avatar_border/4.png',
						showurl: '../../static/image/avatar_border/show_4.png'
					},
					{
						url: '../../static/image/avatar_border/5.png',
						showurl: '../../static/image/avatar_border/show_5.png'
					},
					{
						url: '../../static/image/avatar_border/6.png',
						showurl: '../../static/image/avatar_border/6.png'
					}],
				ctx: null,	// 画布上下文
				canvasId: 'canvas',	// 画布id
				canvas: {
					width: 0,
					height: 0,
					background: 'rgb(255,255,255)'
				},	// 画布的属性
				height: 0,	// 用户手机屏幕高度
				lastX: 0,     // 滑动开始x轴位置
				lastY: 0,     // 滑动开始y轴位置
				slide: null,	// 滑动动画
				back: '',	// 若有头像参数返回这个页面时应用的样式
				photoData: {	// 照片的大小
					width: 0,
					height: 0
				},
				frameData: {	// 头像框的大小
					width: 0,
					height: 0
				},
				move: {
					complete: true,
					dir: ''
				},
				onBottom: false
			}
		},
		methods: {
			chooseImage: function() {
				// 上传图片
				uni.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'],
					sourceType: ['album', 'camera'],
					success: (res) => {
						this.uploadImg = res.tempFilePaths[0];
						// 到剪辑页面进行剪辑
						uni.redirectTo({
							url: `../upload/upload?src=${this.uploadImg}&i=${this.i}`
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
				// 生成头像
				new Promise((resolve, reject) => {
					// 获取头像大小
					uni.getImageInfo({
						src: this.uploadImg,
						success: (res) => {
							resolve(res)
						}
					})
				})
				.then((res) => {
					this.photoData.width = res.width
					this.photoData.height = res.height
					return new Promise((resolve, reject) => {
						// 获取头像框大小
						uni.getImageInfo({
							src: this.currentBorderImg.url,
							success: (res) => {
								resolve(res)
							}
						})
					})
				})
				.then((res) => {
					this.frameData.width = res.width
					this.frameData.height = res.height
					this.canvas.width = res.width	// 画布宽为头像框的宽度
					this.canvas.height = res.height	// 画布高为头像框的高度
					const widthRatio = this.photoData.width/this.frameData.width
					const heightRatio = this.photoData.height/this.frameData.height
					
					
					this.ctx = uni.createCanvasContext(this.canvasId)
					// 先画头像
					if(widthRatio < heightRatio){
					  this.ctx.drawImage(this.uploadImg, 0, 0, photoData.width, this.canvas.height*widthRatio, 0, 0, this.canvas.width, this.canvas.height)
					} else {
					  this.ctx.drawImage(this.uploadImg, 0, 0, this.canvas.width*heightRatio, this.photoData.height, 0, 0, this.canvas.width, this.canvas.height)
					}
					// 再画头像框
					this.ctx.drawImage(this.currentBorderImg.url, 0, 0, this.frameData.width, this.frameData.height, 0, 0, this.canvas.width, this.canvas.height)
					this.ctx.draw()
					setTimeout(() => {
						uni.canvasToTempFilePath({
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
					}, 100)
				})
			},
			handletouchmove: function (event) {
				let currentX = event.touches[0].pageX
				let currentY = event.touches[0].pageY
				let tx = currentX - this.lastX
				let ty = currentY - this.lastY
				// 左右方向滑动
				if (Math.abs(tx) > Math.abs(ty)) {
					if (tx < 0)	// 左滑
						this.move.dir = 'left'
					else if (tx > 0)	// 右滑
						this.move.dir = 'right'
				}
				// 上下方向滑动
				else {
					if (ty < 0) {
						// 下滑
						let animation = uni.createAnimation({
							duration: 1000,
							timingFunction: 'ease'
						})
						this.slide = animation
						animation.translateY(-this.height/2 - 30).step()
						this.slide = animation.export()
						this.onBottom = true
					}
					else if (ty > 0) {
						// 上滑
						let animation = uni.createAnimation({
							duration: 1000,
							timingFunction: 'ease'
						})
						this.slide = animation
						animation.translateY(0).step()
						this.slide = animation.export()
						this.onBottom = false
					}
				}

				//将当前坐标进行保存以进行下一次计算
				this.lastX = currentX
				this.lastY = currentY
			},

			//滑动开始事件
			handletouchtart: function (event) {
				this.lastX = event.touches[0].pageX
				this.lastY = event.touches[0].pageY
				this.move.complete = false
			},
			//滑动结束事件
			handletouchend: function (event) {
				this.move.complete = true
				if(this.move.dir && this.onBottom === false)
					this.switchBorder(this.move.dir)
				this.move.dir = ''
			},
			switchBorder: function(dir) {
				if(dir === 'left') {
					this.i++
					this.currentBorderImg = this.borderImgs[(this.i + 2) % this.borderImgs.length]
				} else {
					this.i === 0 ? this.i = this.borderImgs.length - 1 : this.i--
					this.currentBorderImg = this.borderImgs[(this.i + 2) % this.borderImgs.length]
				}
			}
		},
		onLoad: function(option) {
			uni.showShareMenu({
				withShareTicket: true
			})
			uni.getSystemInfo({
				success: (res) => {
					this.height = res.windowHeight
				}
			})
			// 给当前头像框一个默认值
			if(option.i){
				this.i = +option.i
				this.currentBorderImg = this.borderImgs[(+option.i + 2) % this.borderImgs.length]
			} else {
				this.currentBorderImg = this.borderImgs[this.i + 2]
			}
			// 当有头像参数时获取头像
			if(option.avatar)
				this.uploadImg = option.avatar
			// 判断是否有头像，有头像则定位到页面下半部分
			if(this.uploadImg) {
				this.back = 'translateY(-' + (this.height/2 + 30) + 'px)'
			}
		},
		onShareAppMessage: function() {
			return {
				title: 'GCU生日快乐-校庆头像制作',
				path: '/pages/index/index',
				desc: '和我一起制作头像框，为华广庆生！'
			}
		}
	}
</script>

<style>
/* 	@font-face {
		font-family: 'AdobeFangsongStd-Regular';
		src: url('https://file-web.oss-cn-shenzhen.aliyuncs.com/2020xiaoqing/AdobeFangsongStd-Regular.otf');
	} */
	page {
		margin: 0;
		padding: 0;
		/* font-family: 'AdobeFangsongStd-Regular'; */
	}
	.container {
		overflow: hidden;
	}
	.tip {
		position: absolute;
		width: 428rpx;
		top: 15.7%;
		margin-left: 50%;
		transform: translateX(-50%);
		color: #FFFFFF;
		text-align: center;
	}
	.boderlist {
		position: relative;
		top: 15%;
	}
	.part_top {
		background-image: url('https://file-web.oss-cn-shenzhen.aliyuncs.com/2020xiaoqing/select_background.png');
		background-size: 100% 100%;
	}
	.border {
		width: 145rpx;
		height: 145rpx;
		background-color: #FFFFFF;
		border-radius: 50%;
		position: absolute;
	}
	.border image {
		width: 145rpx;
		height: 145rpx;
	}
	.border_left {
		left: 0;
		transform: translateX(-50%);
	}
	.border_right {
		right: 0;
		transform: translateX(50%);
	}
	.border_left2 {
		top: 193rpx;
		left: 68rpx;
	}
	.border_right2 {
		top: 193rpx;
		right: 68rpx;
	}
	.border_main {
		background-color: #FFFFFF;
		border-radius: 50%;
		position: fixed;
		top: 35%;
		left: 50%;
		transform: translateX(-50%);
		width: 220rpx;
		height: 220rpx;
		z-index: 99999999999999;
	}
	.user_avatar {
		width: 220rpx;
		height: 220rpx;
		border-radius: 50%;
	}
	.user_avatar_border {
		position: absolute;
		top: 0;
		left: 0;
		width: 220rpx;
		height: 220rpx;
	}
	.btn {
		width: 247rpx;
		margin-left: 50%;
		transform: translateX(-50%);
		margin-top: 230rpx;
	}
	.btn-choose {
		height: 65rpx;
		width: 247rpx;
		background-color: #a2d4fc;
		line-height: 65rpx;
		border-radius: 20rpx;
		text-align: center;
	}
	.btn-generate {
		margin-top: 49rpx;
		background-color: #f3b4e1;
		height: 65rpx;
		width: 247rpx;
		line-height: 65rpx;
		border-radius: 20rpx;
		text-align: center;
	}
	.boldText {
		font-weight: bold;
	}
</style>
