<template>
	<view class="mbe-page">
		<view class="mbe-card info-card">
			<view class="info-row">
				<text class="info-item">✨ {{ points }}</text>
				<text class="info-item">🍖 {{ food }}</text>
				<text class="info-item">🪙 {{ coins }}</text>
			</view>
		</view>

		<view class="mbe-card egg-card">
			<text class="section-title">🥚 当前孵化</text>
			<view class="egg-display">
				<text class="egg-emoji">{{ eggEmoji }}</text>
				<view class="egg-info">
					<text class="egg-name">{{ currentEgg.name }}</text>
					<view class="progress-bar">
						<view class="progress-fill" :style="{ width: currentEgg.progress + '%' }"></view>
					</view>
					<text class="egg-percent">进度 {{ currentEgg.progress }}%</text>
				</view>
			</view>
			<view v-if="currentEgg.progress >= 100" class="mbe-btn hatch-btn" @click="hatch">
				<text class="btn-text">✨ 破壳</text>
			</view>
			<text v-else class="egg-tip">每答对一题 +10% 进度，去学习页答题吧～</text>
		</view>

		<view class="mbe-card animals-card">
			<text class="section-title">🐾 已拥有动物（点喂养消耗 1 🍖）</text>
			<view class="animals-grid">
				<view
					v-for="(a, i) in animals"
					:key="i"
					class="animal-cell"
					:class="stageClass(a.stage)"
				>
					<view class="animal-emoji-wrap" :class="stageClass(a.stage)">
						<text class="animal-emoji">{{ a.emoji }}</text>
						<text v-if="a.stage === '老年'" class="crown">👑</text>
						<text v-if="a.stage === '成年'" class="sparkle">✨</text>
					</view>
					<text class="animal-name">{{ a.name }}</text>
					<text class="animal-stage-tag">{{ a.stage }}</text>
					<view class="growth-bar">
						<view class="growth-fill" :style="{ width: (a.growth / 5 * 100) + '%' }"></view>
					</view>
					<text class="growth-text">成长 {{ a.growth }}/5</text>
					<view
						class="mbe-btn feed-btn"
						:class="{ 'disabled': food <= 0 }"
						@click="feedAnimal(i)"
					>
						<text class="btn-text">🍖 喂养</text>
					</view>
				</view>
			</view>
			<view v-if="animals.length === 0" class="empty">
				<text class="empty-text">还没有动物～去扭蛋页抽一只吧！</text>
				<view class="mbe-btn goto-gacha-btn" @click="goGacha">
					<text class="btn-text">去扭蛋</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const STAGES = ['幼年', '青年', '成年', '老年'];
	const FALLBACK_POOL = [
		{ name: '鹌鹑', emoji: '🐦' },
		{ name: '小鸡', emoji: '🐤' },
		{ name: '小鸭', emoji: '🦆' },
		{ name: '小松鼠', emoji: '🐿️' }
	];

	export default {
		data() {
			return {
				points: 0,
				food: 0,
				coins: 0,
				animals: [],
				currentEgg: { name: '未知的蛋', progress: 0 }
			};
		},
		computed: {
			eggEmoji() {
				return this.currentEgg.progress >= 100 ? '✨🥚✨' : '🥚';
			}
		},
		onShow() {
			this.loadState();
		},
		methods: {
			loadState() {
				this.points = uni.getStorageSync('points') || 0;
				this.food = uni.getStorageSync('food') || 0;
				this.coins = uni.getStorageSync('coins') || 0;
				const a = uni.getStorageSync('animals');
				this.animals = Array.isArray(a) ? a : [];
				const e = uni.getStorageSync('currentEgg');
				this.currentEgg = (e && typeof e === 'object') ? e : { name: '未知的蛋', progress: 0 };
			},
			stageClass(stage) {
				return 'stage-' + stage;
			},
			hatch() {
				let animalName, emoji;
				if (this.currentEgg.animalName && this.currentEgg.emoji) {
					animalName = this.currentEgg.animalName;
					emoji = this.currentEgg.emoji;
				} else {
					const pick = FALLBACK_POOL[Math.floor(Math.random() * FALLBACK_POOL.length)];
					animalName = pick.name;
					emoji = pick.emoji;
				}
				const newAnimal = {
					name: animalName,
					emoji: emoji,
					stage: '幼年',
					growth: 0
				};
				this.animals.push(newAnimal);
				uni.setStorageSync('animals', this.animals);
				this.currentEgg = { name: '未知的蛋', progress: 0 };
				uni.setStorageSync('currentEgg', this.currentEgg);

				uni.showModal({
					title: '🎉 破壳成功！',
					content: `恭喜获得 ${emoji} ${animalName}！\n已加入你的领地。`,
					showCancel: false
				});
			},
			feedAnimal(i) {
				if (this.food <= 0) {
					uni.showToast({ title: '🍖 不够，去学习赚积分换食物', icon: 'none' });
					return;
				}
				this.food -= 1;
				uni.setStorageSync('food', this.food);

				const animal = this.animals[i];
				animal.growth = (animal.growth || 0) + 1;

				const stageIdx = STAGES.indexOf(animal.stage);
				if (animal.growth >= 5 && stageIdx < STAGES.length - 1) {
					animal.stage = STAGES[stageIdx + 1];
					animal.growth = 0;
					uni.showToast({ title: `${animal.name} 进阶到 ${animal.stage}！`, icon: 'none' });
				} else if (animal.growth >= 5 && stageIdx === STAGES.length - 1) {
					animal.growth = 5;
					uni.showToast({ title: `${animal.name} 已是老年，满级！`, icon: 'none' });
				} else {
					uni.showToast({ title: '+1 成长值', icon: 'none' });
				}

				uni.setStorageSync('animals', this.animals);
			},
			goGacha() {
				uni.switchTab({ url: '/pages/gacha/gacha' });
			}
		}
	};
</script>

<style scoped>
	.info-card {
		background: #FFD93D;
	}
	.info-row {
		display: flex;
		justify-content: space-around;
	}
	.info-item {
		font-size: 30rpx;
		font-weight: 800;
		color: #333;
	}

	.egg-card {
		background: #4ECDC4;
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}
	.section-title {
		font-size: 36rpx;
		font-weight: 800;
		color: #333;
	}
	.egg-display {
		display: flex;
		align-items: center;
		gap: 30rpx;
	}
	.egg-emoji {
		font-size: 130rpx;
	}
	.egg-info {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 10rpx;
	}
	.egg-name {
		font-size: 30rpx;
		font-weight: 800;
		color: #333;
	}
	.progress-bar {
		width: 100%;
		height: 26rpx;
		background: #FFF;
		border: 3rpx solid #333;
		border-radius: 13rpx;
		overflow: hidden;
	}
	.progress-fill {
		height: 100%;
		background: #FFD93D;
		transition: width 0.3s;
	}
	.egg-percent {
		font-size: 24rpx;
		font-weight: 700;
		color: #333;
	}
	.hatch-btn {
		background: #FF6B9D;
		align-self: center;
		padding: 26rpx 80rpx;
	}
	.hatch-btn .btn-text {
		color: #FFF;
		font-size: 34rpx;
		font-weight: 800;
	}
	.egg-tip {
		font-size: 24rpx;
		color: #333;
		text-align: center;
		font-weight: 600;
	}

	.animals-card {
		background: #B5E85C;
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}
	.animals-grid {
		display: flex;
		flex-wrap: wrap;
		gap: 20rpx;
	}
	.animal-cell {
		background: #FFF3C4;
		border: 4rpx solid #333;
		border-radius: 18rpx;
		box-shadow: 6rpx 6rpx 0 #333;
		padding: 20rpx;
		width: 100%;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 8rpx;
	}
	.animal-emoji-wrap {
		position: relative;
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}
	.animal-emoji {
		display: inline-block;
	}
	.stage-幼年 .animal-emoji {
		font-size: 60rpx;
		transform: scale(0.8);
	}
	.stage-青年 .animal-emoji {
		font-size: 76rpx;
	}
	.stage-成年 .animal-emoji {
		font-size: 92rpx;
	}
	.stage-老年 .animal-emoji {
		font-size: 92rpx;
	}
	.crown {
		position: absolute;
		top: -22rpx;
		right: -16rpx;
		font-size: 36rpx;
	}
	.sparkle {
		position: absolute;
		top: -18rpx;
		right: -12rpx;
		font-size: 32rpx;
	}
	.animal-name {
		font-size: 28rpx;
		font-weight: 800;
		color: #333;
	}
	.animal-stage-tag {
		font-size: 22rpx;
		color: #FFF;
		background: #FF6B9D;
		padding: 4rpx 14rpx;
		border: 2rpx solid #333;
		border-radius: 12rpx;
		font-weight: 700;
	}
	.growth-bar {
		width: 80%;
		height: 18rpx;
		background: #FFF;
		border: 2rpx solid #333;
		border-radius: 9rpx;
		overflow: hidden;
	}
	.growth-fill {
		height: 100%;
		background: #95E065;
		transition: width 0.3s;
	}
	.growth-text {
		font-size: 22rpx;
		color: #555;
		font-weight: 700;
	}
	.feed-btn {
		background: #FFD93D;
		padding: 18rpx 40rpx;
		margin-top: 8rpx;
	}
	.feed-btn .btn-text {
		color: #333;
		font-size: 28rpx;
		font-weight: 800;
	}
	.feed-btn.disabled {
		background: #D9D9D9;
		opacity: 0.7;
	}

	.empty {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 16rpx;
		padding: 30rpx;
	}
	.empty-text {
		font-size: 26rpx;
		color: #555;
		font-weight: 600;
	}
	.goto-gacha-btn {
		background: #FF6B9D;
	}
	.goto-gacha-btn .btn-text {
		color: #FFF;
		font-weight: 800;
	}
</style>
