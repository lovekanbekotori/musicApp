<template>
	<view>
		<view class="status_bar">
			<!-- 这里是状态栏 -->
		</view>
		<view class="ml-3 mt-2 position-fixed" hover-class="animated jello" @tap="back">
			<text class="text-white iconfont iconfanhui-copy-copy font-lg font-weight-bold"></text>
		</view>
		<view class="flex justify-center align-center mt-3 w-100" style="height: 100rpx;">
			<view class="rounded-circle px-2 mx-1 flex align-center" :class="tabIndex===0?'text-white border':'text-light-muted'"
			 style="height: 70rpx;" @tap="chageTab(0)">音乐</view>
			<view class="rounded-circle px-2 mx-1 flex align-center" :class="tabIndex===1?'text-white border':'text-light-muted'"
			 style="height: 70rpx;" @tap="chageTab(1)">歌词</view>
		</view>
		<swiper class="swiper" :duration="300" :current="tabIndex" :style="{height: scrollH+'px'}" @change="onChangeTab">
			<swiper-item>
				<view>
					<view class="mt-2">
						<!-- 显示图片 -->
						<!-- <image :src="Music.img" style="width: 750rpx;height: 425rpx;" mode="aspectFill"> -->
						<image :src="coverPath" style="width: 750rpx;height: 750rpx;" mode="aspectFill">
					</view>
					<view class="fixed-bottom" style="background-color: #131313;height: 500rpx;width: 750rpx;">
						<view style="height: 20rpx;"></view>
						<!-- 文字滚动 -->
						<view class="m-2 rounded">
							<uni-notice-bar style="margin-bottom: 100rpx;" size="50rpx" background-color="#131313" color="#FFFFFF"
							 :scrollable="getPlaying" :speed="30" single="true" :text="Music.title+'        '+Music.author"></uni-notice-bar>
						</view>
						<view class="flex justify-center flex-column">
							<!-- 播放滑动条 -->
							<slider backgroundColor="#434343" :max="music.max" :value="music.played" block-size="12" activeColor="#FFFFFF"
							 style="width: 660rpx;" @changing="sliderChanging" @change="sliderChanged"></slider>
							<view class="flex justify-between px-5" style="margin-top: -10px;">
								<text class="text-light-muted font-sm">{{formatTime1}}</text>
								<text class="text-light-muted font-sm">{{formatTime2}}</text>
							</view>
							<!-- <view style="height: 50rpx;"></view> -->
							<!-- 播放按钮等 -->
							<view class="flex flex-row align-center justify-center">
								<!-- 播放模式-->
								<view class="text-white iconfont mx-4 font-weight-bolder" style="font-size: 60rpx;" :class="playMode"
								 hover-class="animated pulse" @tap="changePlayMode"></view>
								<!-- 上一首-->
								<view class="text-white iconfont iconshangyishou mx-4" style="font-size: 75rpx;" hover-class="animated pulse"
								 @tap="lastMusic"></view>
								<!-- 播放 -->
								<view @tap="changeState" class="text-white iconfont mx-4" style="font-size: 70rpx;" hover-class="animated pulse"
								 :class="getPlaying ? 'iconzantingtingzhi':'iconziyuanldpi11'"></view>
								<!-- 下一首 -->
								<view class="text-white iconfont iconxiayishou  mx-4" style="font-size: 75rpx;" hover-class="animated pulse"
								 @tap="nextMusic"></view>
								<!-- 分享 -->
								<view class="text-white font-lger iconfont iconfenxiang mx-4" hover-class="animated pulse" @tap="share"></view>
							</view>
						</view>
					</view>
				</view>
			</swiper-item>
			<swiper-item>
				歌词界面
			</swiper-item>
		</swiper>

	</view>
</template>

<script>
	import uniNoticeBar from '@/components/uni-notice-bar/uni-notice-bar.vue';
	import service from '@/service.js';
	import $T from '@/common/time.js';
	import {
		mapGetters,
		mapMutations,
		mapState
	} from "vuex";

	let timer;
	export default {
		components: {
			uniNoticeBar
		},
		data() {
			return {
				scrollH: 500,
				tabIndex: 0,
				music: {
					max: 0, //播放总时长
					played: 0, //已播放时长
				},
				coverPath: this.show_img()
			}
		},
		computed: {
			...mapGetters(['getPopState', 'getPlaying', 'getPlayMode']),
			...mapState(['Audio', 'Music', 'hasLogin', 'playing']),
			formatTime1() {
				return $T.formatSeconds(this.music.played);
			},
			formatTime2() {
				return $T.formatSeconds(this.music.max);
			},
			//播放模式
			playMode() {
				// console.log(this.getPlayMode);
				switch (this.getPlayMode) {
					case 1:
						return 'iconliebiaoxunhuan';
					case 2:
						return 'icondanquxunhuan';
					case 3:
						return 'iconsuiji';
				}
				return 'iconliebiaoxunhuan';
			}
		},
		onLoad(e) {
			//初始化高度
			uni.getSystemInfo({
				success: res => {
					//console.log(res);
					this.scrollH = res.windowHeight - res.statusBarHeight - uni.upx2px(130);
				}
			})

			let that = this; //传递自身this
			//检查wifi或数据网络
			// uni.getNetworkType({
			// 	success: function(res) {
			// 		if (res.networkType !== "wifi" && res.networkType !== 'none' && service.getSetting().onlyUseWifi || res.networkType !==
			// 			"wifi" && service.getSetting().onlyUseWifi === undefined && res.networkType !== 'none') {
			// 			console.log("当前非wifi状态 使用流量播放");
			// 			uni.showModal({
			// 				title: '提示',
			// 				content: '继续将使用流量播放',
			// 				confirmText: '继续',
			// 				success: (res) => {
			// 					//取消返回上一个页面
			// 					if (res.cancel) {
			// 						that.$store.commit("setPopState", false); //关闭音乐弹出层
			// 						this.setMusic(undefined); //音乐数据置空
			// 						uni.navigateBack({
			// 							delta: 1
			// 						})
			// 					} else {
			// 						//使用流量播放
			// 						service.setSetting('onlyUseWifi', false);
			// 					}
			// 				}
			// 			})
			// 		}
			// 	},
			// });
			//this.$store.commit("setPopState", true); //可用mutations方法代替  显示音乐弹层
			this.setPopState(true);
			if (JSON.stringify(this.Music) !== '{}' && this.Music != undefined && this.Audio.src !== this.Music.src) {
				//播放音乐登录检查
				uni.$emit('playCheck');
				console.log("音乐页面");

			}
			//设置播放进度监听
			timer = setInterval(() => {
				//console.log("音乐播放页监听");
				this.music.max = this.Audio.duration; //音乐总时长
				this.music.played = this.Audio.currentTime;
			}, 500)

		},
		onUnload() {
			console.log("播放页面卸载");
			clearInterval(timer);
		},
		methods: {
			...mapMutations(['setPopState', 'setPlaying', 'setMusic', 'setPlayMode']),
			//随机获取一张图片
			show_img() {
				let num = this.randomNum(1, 20);
				return '/static/show_img/' + num + '.jpg'
			},
			//上一首歌曲
			lastMusic() {

				uni.request({
					url: service.DOMAIN + 'api/v1.Music/lastMusic',
					method: 'POST',
					data: {
						id: this.Music.id
					},
					success: (res) => {
						if (res.data.data != null) {
							this.coverPath = this.show_img();
							this.setMusic(res.data.data)
						} else {
							plus.nativeUI.toast('没有更多歌曲了');
						}
					}
				});
			},
			//下一首歌曲
			nextMusic() {
				this.coverPath = this.show_img();
				uni.request({
					url: service.DOMAIN + 'api/v1.Music/nextMusic',
					method: 'POST',
					data: {
						id: this.Music.id
					},
					success: (res) => {
						if (res.data.data != null) {
							this.coverPath = this.show_img();
							this.setMusic(res.data.data)
						} else {
							plus.nativeUI.toast('没有更多歌曲了');
						}
					}
				});
			},
			//改变播放模式
			changePlayMode() {
				this.setPlayMode(this.getPlayMode + 1);
				if (this.getPlayMode > 3) this.setPlayMode(1);
				switch (this.getPlayMode) {
					case 1:
						uni.showToast({
							title: '切换到列表播放'
						});
						break;
					case 2:
						uni.showToast({
							title: '切换到单曲循环'
						});
						break;
					case 3:
						uni.showToast({
							title: '切换到随机播放'
						});
						break;
				}

			},
			//生成从minNum到maxNum的随机数
			randomNum(minNum, maxNum) {
				switch (arguments.length) {
					case 1:
						return parseInt(Math.random() * minNum + 1, 10);
						break;
					case 2:
						return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10);
						break;
					default:
						return 0;
						break;
				}
			},
			//swiper滑动事件
			onChangeTab(e) {
				this.tabIndex = e.detail.current;
			},
			//更改tab面板
			chageTab(index) {
				this.tabIndex = index;
			},
			//返回上一个页面
			back() {
				uni.navigateBack({
					delta: 1
				})
			},
			//音乐方波进度变化事件
			// onTimeUpdate() {
			// 	//this.$mp.page.$getAppWebview().id 
			// 	//console.log("音乐播放页:" + this.Audio.currentTime);
			// 	this.music.max = this.Audio.duration; //音乐总时长
			// 	this.music.played = this.Audio.currentTime;
			// },
			//拖动中播放进度条变化
			sliderChanging(e) {
				this.music.played = e.detail.value;
			},
			//拖动进度条完成后事件
			sliderChanged(e) {
				this.Audio.seek(e.detail.value);
			},
			//点击暂停播放
			changeState() {
				console.log('改变播放状态')
				if (this.getPlaying) {
					//暂停播放
					this.Audio.pause();
					this.setPlaying(false);
				} else {
					//继续播放
					this.Audio.play();
					this.setPlaying(true);
				}
			},
			//分享歌曲
			share() {
				console.log("分享歌曲");
				uni.share({
					provider: 'weixin',
					scene: "WXSenceTimeline",
					type: 3,
					imageUrl: this.Music.cover,
					title: this.Music.title,
					mediaUrl: this.Music.src,
					success: ret => {
						//点击分享回调
						console.log("分享点击回调")
						console.log(JSON.stringify(ret));
					},
					fail: function(err) {
						console.log("fail:" + JSON.stringify(err));
					}
				});
			}
		}
	}
</script>

<style>
	page {
		background-color: #000000;
	}
</style>
