<template>
	<view class="content">
		<button @click="toSelect">to select</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
			}
		},
		onLoad() {

		},
		methods: {
			toSelect: function() {
				uni.navigateTo({
					url: '../select/select'
				})
			}
		}
	}
</script>

<style>
	.content {
		background-image: url(../../static/image/background.png);
		background-size: 100% 100%;
	}
</style>
