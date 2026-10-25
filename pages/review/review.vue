<template>
	<view class="mbe-page">
		<text class="page-title">📚 错词复习</text>
		<text class="page-sub">每记住一个 +2 积分（双倍奖励）</text>

		<!-- 空状态 -->
		<view v-if="wrongWords.length === 0" class="mbe-card empty-state">
			<text class="empty-icon">🎉</text>
			<text class="empty-title">太棒了！暂无错词，继续保持哦~</text>
			<view class="mbe-btn back-btn" @click="goLearn">
				<text class="btn-text">📖 返回学习</text>
			</view>
		</view>

		<!-- 复习卡片 -->
		<view v-else class="mbe-card review-card" :class="{ 'is-back': showBack }">
			<view class="counter-row">
				<text class="card-counter">{{ currentIdx + 1 }} / {{ wrongWords.length }}</text>
				<text class="card-tag">{{ current.source || 'CET-4' }}</text>
			</view>

			<!-- 正面：单词 + 音标 -->
			<view v-if="!showBack" class="card-face card-front">
				<text class="card-word">{{ current.word }}</text>
				<view class="meta-row">
					<text v-if="current.phonetic" class="card-phonetic">[{{ current.phonetic }}]</text>
					<text v-if="current.partOfSpeech" class="card-pos">{{ current.partOfSpeech }}</text>
				</view>
				<text class="card-hint">🤔 想想这个词的中文意思…</text>
				<view class="mbe-btn flip-btn" @click="flipToBack">
					<text class="btn-text">📖 查看释义</text>
				</view>
			</view>

			<!-- 背面：释义 + 例句 + 动作 -->
			<view v-else class="card-face card-back">
				<view class="back-head">
					<text class="back-word">{{ current.word }}</text>
					<text v-if="current.phonetic" class="back-phonetic">[{{ current.phonetic }}]</text>
				</view>
				<text class="back-translation">✅ {{ current.translation }}</text>
				<view v-if="current.example" class="example-block">
					<text class="example-en">📝 {{ current.example }}</text>
					<text v-if="current.exampleZh" class="example-zh">📖 {{ current.exampleZh }}</text>
				</view>
				<view v-if="current.extension" class="extension-block">
					<text class="extension-text">💡 {{ current.extension }}</text>
				</view>
				<view class="action-row">
					<view class="mbe-btn forget-btn" @click="forgetWord">
						<text class="btn-text">🔄 没记住</text>
					</view>
					<view class="mbe-btn remember-btn" @click="rememberWord">
						<text class="btn-text">✨ 记住了 (+2)</text>
					</view>
				</view>
			</view>
		</view>

		<!-- 底部退出 -->
		<view class="mbe-btn exit-btn" @click="goLearn">
			<text class="btn-text">← 返回学习</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wrongWords: [],
				currentIdx: 0,
				showBack: false
			};
		},
		computed: {
			current() {
				return this.wrongWords[this.currentIdx] || {};
			}
		},
		onShow() {
			this.loadWrongWords();
		},
		methods: {
			loadWrongWords() {
				const ww = uni.getStorageSync('wrongWords');
				this.wrongWords = Array.isArray(ww) ? ww : [];
				if (this.currentIdx >= this.wrongWords.length) {
					this.currentIdx = 0;
				}
				this.showBack = false;
			},
			flipToBack() {
				this.showBack = true;
			},
			// 没记住：重置为正面，并把该词排到队列末尾
			forgetWord() {
				const cur = this.wrongWords[this.currentIdx];
				if (!cur) return;
				this.wrongWords.splice(this.currentIdx, 1);
				this.wrongWords.push(cur);
				uni.setStorageSync('wrongWords', this.wrongWords);
				// 若原位置是末尾，则包裹回开头；否则保持 currentIdx 自然指向下一词
				if (this.currentIdx >= this.wrongWords.length - 1) {
					this.currentIdx = 0;
				}
				this.showBack = false;
				uni.showToast({ title: '再来一次！', icon: 'none' });
			},
			// 记住了：从错题本移除 + 总积分 +2
			rememberWord() {
				if (this.wrongWords.length === 0) return;
				this.wrongWords.splice(this.currentIdx, 1);
				uni.setStorageSync('wrongWords', this.wrongWords);
				const pts = (uni.getStorageSync('points') || 0) + 2;
				uni.setStorageSync('points', pts);
				if (this.currentIdx >= this.wrongWords.length) {
					this.currentIdx = 0;
				}
				this.showBack = false;
				uni.showToast({ title: '+2 积分！', icon: 'none' });
			},
			goLearn() {
				uni.navigateBack({ delta: 1 });
			}
		}
	};
</script>

<style scoped>
	.page-title {
		font-size: 44rpx;
		font-weight: 900;
		color: #333;
		text-align: center;
	}
	.page-sub {
		font-size: 24rpx;
		color: #5D4037;
		font-weight: 700;
		text-align: center;
		margin-top: -8rpx;
	}

	/* 空状态 */
	.empty-state {
		background: linear-gradient(180deg, #FFF9E6 0%, #FFD93D 100%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 24rpx;
		padding: 80rpx 40rpx;
	}
	.empty-icon {
		font-size: 140rpx;
		animation: bob 1.6s ease-in-out infinite;
	}
	@keyframes bob {
		0%, 100% { transform: translateY(0); }
		50% { transform: translateY(-12rpx); }
	}
	.empty-title {
		font-size: 30rpx;
		font-weight: 800;
		color: #333;
		text-align: center;
		line-height: 1.5;
	}
	.back-btn {
		background: #FF6B9D;
		padding: 22rpx 60rpx;
	}
	.back-btn .btn-text {
		color: #FFF;
		font-size: 30rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}

	/* 复习卡片 */
	.review-card {
		background: #FFF9E6;
		display: flex;
		flex-direction: column;
		gap: 18rpx;
		padding: 40rpx 32rpx;
		position: relative;
	}
	.counter-row {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	.card-counter {
		font-size: 26rpx;
		font-weight: 800;
		color: #5D4037;
		background: #FFF;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 6rpx 18rpx;
	}
	.card-tag {
		font-size: 22rpx;
		color: #FFF;
		background: #4ECDC4;
		border: 2rpx solid #333;
		border-radius: 10rpx;
		padding: 4rpx 14rpx;
		font-weight: 700;
	}

	.card-face {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 18rpx;
		animation: face-in 0.3s ease-out;
	}
	@keyframes face-in {
		0% { opacity: 0; transform: scale(0.92); }
		100% { opacity: 1; transform: scale(1); }
	}

	/* 正面 */
	.card-front {
		padding: 60rpx 20rpx 40rpx;
	}
	.card-word {
		font-size: 96rpx;
		font-weight: 900;
		color: #333;
		letter-spacing: 2rpx;
	}
	.meta-row {
		display: flex;
		align-items: baseline;
		gap: 16rpx;
		flex-wrap: wrap;
		justify-content: center;
	}
	.card-phonetic {
		font-size: 30rpx;
		color: #5D4037;
		font-style: italic;
		font-weight: 700;
	}
	.card-pos {
		font-size: 24rpx;
		color: #FFF;
		background: #4ECDC4;
		border: 2rpx solid #333;
		border-radius: 10rpx;
		padding: 4rpx 14rpx;
		font-weight: 700;
	}
	.card-hint {
		font-size: 26rpx;
		color: #795548;
		font-weight: 600;
	}
	.flip-btn {
		background: #FFD93D;
		padding: 24rpx 70rpx;
		margin-top: 16rpx;
	}
	.flip-btn .btn-text {
		color: #333;
		font-size: 32rpx;
		font-weight: 800;
	}

	/* 背面 */
	.card-back {
		padding: 30rpx 10rpx 20rpx;
		gap: 16rpx;
	}
	.back-head {
		display: flex;
		align-items: baseline;
		gap: 16rpx;
		flex-wrap: wrap;
		justify-content: center;
	}
	.back-word {
		font-size: 56rpx;
		font-weight: 900;
		color: #FF6B9D;
		text-shadow: 2rpx 2rpx 0 #FFF;
	}
	.back-phonetic {
		font-size: 26rpx;
		color: #5D4037;
		font-style: italic;
		font-weight: 700;
	}
	.back-translation {
		font-size: 36rpx;
		font-weight: 900;
		color: #333;
		text-align: center;
		background: #FFF;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 16rpx 24rpx;
		width: 100%;
		box-sizing: border-box;
	}
	.example-block {
		background: #FFF;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 18rpx 22rpx;
		display: flex;
		flex-direction: column;
		gap: 10rpx;
		width: 100%;
		box-sizing: border-box;
	}
	.example-en {
		font-size: 28rpx;
		font-weight: 700;
		color: #5D4037;
	}
	.example-zh {
		font-size: 26rpx;
		color: #795548;
	}
	.extension-block {
		background: #E1F5FE;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 16rpx 22rpx;
		width: 100%;
		box-sizing: border-box;
	}
	.extension-text {
		font-size: 26rpx;
		color: #0D47A1;
		font-weight: 600;
		line-height: 1.5;
	}

	/* 动作按钮 */
	.action-row {
		display: flex;
		gap: 20rpx;
		margin-top: 12rpx;
	}
	.forget-btn {
		flex: 1;
		background: #FFB5D8;
		padding: 22rpx 0;
	}
	.forget-btn .btn-text {
		color: #333;
		font-size: 30rpx;
		font-weight: 800;
	}
	.remember-btn {
		flex: 1;
		background: linear-gradient(90deg, #95E065 0%, #66BB6A 100%);
		padding: 22rpx 0;
	}
	.remember-btn .btn-text {
		color: #FFF;
		font-size: 30rpx;
		font-weight: 900;
		text-shadow: 2rpx 2rpx 0 #333;
	}

	/* 退出按钮 */
	.exit-btn {
		background: #FFF;
		padding: 22rpx 0;
		margin-top: 12rpx;
	}
	.exit-btn .btn-text {
		color: #333;
		font-size: 28rpx;
		font-weight: 800;
	}
</style>
