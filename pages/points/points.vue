<template>
	<view class="mbe-page">
		<!-- 顶部三栏状态栏 -->
		<view class="top-bar">
			<view class="mbe-card stat-card stat-points">
				<text class="stat-emoji">✨</text>
				<text class="stat-value">{{ points }}</text>
				<text class="stat-label">积分</text>
			</view>
			<view class="mbe-card stat-card stat-food">
				<text class="stat-emoji">🍖</text>
				<text class="stat-value">{{ food }}</text>
				<text class="stat-label">食物</text>
			</view>
			<view class="mbe-card stat-card stat-coins">
				<text class="stat-emoji">🪙</text>
				<text class="stat-value">{{ coins }}</text>
				<text class="stat-label">游戏币</text>
			</view>
		</view>

		<!-- 兑换所横幅 -->
		<view class="mbe-card banner-btn" @click="openExchange">
			<text class="banner-text">🏪 兑换所</text>
			<text class="banner-sub">用积分换食物 / 游戏币</text>
		</view>

		<!-- 扭蛋机 2x2 立体版 -->
		<view class="mbe-card machines-card">
			<text class="section-title">🎰 扭蛋机</text>
			<view class="machines">
				<view
					v-for="(m, key) in machines"
					:key="key"
					class="machine"
					:class="'machine-' + key"
				>
					<!-- 玻璃球 + 金属底座 -->
					<view class="machine-body">
						<view class="glass-dome">
							<text class="dome-content">🔮</text>
							<view class="dome-highlight"></view>
							<view class="dome-highlight-small"></view>
						</view>
						<view class="metal-base">
							<view class="coin-slot"></view>
						</view>
					</view>
					<view class="machine-cost-tag">
						<text class="cost-text">🪙 × {{ m.cost }}</text>
					</view>
					<view class="draw-btn" @click="draw(key)">
						<text class="draw-text">投币抽取</text>
					</view>
					<text class="machine-label">{{ m.label }}</text>
					<view class="machine-hint">
						<text v-if="m.isMystery" class="hint-mystery">❓ 神秘大奖</text>
						<view v-else class="hint-list">
							<text
								v-for="(p, pi) in m.pool"
								:key="pi"
								class="hint-item"
							>{{ p.emoji }} {{ p.name }}{{ pi < m.pool.length - 1 ? ' / ' : '' }}</text>
						</view>
					</view>
				</view>
			</view>
		</view>

		<!-- 兑换所弹窗 -->
		<view class="modal-mask" :class="{ show: showExchange }" @click="closeExchange">
			<view class="mbe-card exchange-modal" @click.stop="">
				<view class="close-btn" @click="closeExchange">
					<text class="close-text">×</text>
				</view>
				<text class="modal-title">🏪 兑换所</text>
				<text class="modal-sub">选择一个兑换方案</text>
				<view class="mbe-btn exchange-btn food-exchange" @click="exchange(5, 1, 'food')">
					<text class="btn-text">5 ✨ → 1 🍖</text>
				</view>
				<view class="mbe-btn exchange-btn coin-exchange" @click="exchange(10, 1, 'coins')">
					<text class="btn-text">10 ✨ → 1 🪙</text>
				</view>
				<view class="mbe-btn exchange-btn potion-exchange" @click="exchange(10, 1, 'potions')">
					<text class="btn-text">10 ✨ → 1 💊 药水</text>
				</view>
				<view class="mbe-btn exchange-btn magic-exchange" @click="exchange(20, 1, 'magicPotions')">
					<text class="btn-text">20 ✨ → 1 ✨ 魔法药水</text>
				</view>
			</view>
		</view>

		<!-- 抽取结果弹窗 -->
		<view class="modal-mask" :class="{ show: showResult }" @click="closeResult">
			<view class="mbe-card result-modal" @click.stop="">
				<text class="result-egg">🥚</text>
				<text class="result-title">🎉 恭喜获得</text>
				<text class="result-level-tag">[{{ resultLevel }}]</text>
				<text class="result-title2">{{ resultTitle2 }}</text>
				<text class="result-tip">已加入孵化队列！</text>
				<view class="mbe-btn confirm-btn" @click="closeResult">
					<text class="btn-text">确定</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const MACHINES = {
		junior: {
			label: '初级扭蛋机',
			cost: 3,
			pool: [
				{ name: '小鸡', emoji: '🐤' },
				{ name: '小鸭', emoji: '🦆' },
				{ name: '小松鼠', emoji: '🐿️' }
			]
		},
		middle: {
			label: '中级扭蛋机',
			cost: 5,
			pool: [
				{ name: '小猫', emoji: '🐱' },
				{ name: '小狗', emoji: '🐶' },
				{ name: '小猴', emoji: '🐵' }
			]
		},
		senior: {
			label: '高级扭蛋机',
			cost: 10,
			pool: [
				{ name: '小狐狸', emoji: '🦊' },
				{ name: '小鹿', emoji: '🦌' },
				{ name: '小羊', emoji: '🐑' }
			]
		},
		special: {
			label: '特级扭蛋机',
			cost: 50,
			isMystery: true,
			// 专属图片系隐藏角色：奶蛙
			pool: [
				{
					name: '奶蛙',
					emoji: '👽',
					isSpecialImage: true,
					images: [
						'/static/images/naiba_1.png', // 幼年
						'/static/images/naiba_2.png', // 青年
						'/static/images/naiba_3.png', // 成年
						'/static/images/naiba_4.png'  // 老年
					]
				}
			]
		}
	};

	export default {
		data() {
				return {
					points: 0,
					food: 0,
					coins: 0,
					potions: 0,
					magicPotions: 0,
					machines: MACHINES,
					showExchange: false,
					showResult: false,
					resultLevel: '初级',
					resultTitle2: '神秘的蛋'
				};
			},
			onShow() {
				this.loadState();
			},
			methods: {
				loadState() {
					this.points = uni.getStorageSync('points') || 0;
					this.food = uni.getStorageSync('food') || 0;
					this.coins = uni.getStorageSync('coins') || 0;
					this.potions = uni.getStorageSync('potions') || 0;
					this.magicPotions = uni.getStorageSync('magicPotions') || 0;
				},
			openExchange() {
				this.showExchange = true;
			},
			closeExchange() {
				this.showExchange = false;
			},
			exchange(cost, gain, key) {
				if (this.points < cost) {
					uni.showToast({ title: `积分不足，需要 ${cost} ✨`, icon: 'none' });
					return;
				}
				this.points -= cost;
				if (key === 'food') {
					this.food += gain;
					uni.setStorageSync('food', this.food);
					uni.showToast({ title: `+${gain} 🍖`, icon: 'none' });
				} else if (key === 'coins') {
					this.coins += gain;
					uni.setStorageSync('coins', this.coins);
					uni.showToast({ title: `+${gain} 🪙`, icon: 'none' });
				} else if (key === 'potions') {
					this.potions += gain;
					uni.setStorageSync('potions', this.potions);
					uni.showToast({ title: `+${gain} 💊 药水`, icon: 'none' });
				} else if (key === 'magicPotions') {
					this.magicPotions += gain;
					uni.setStorageSync('magicPotions', this.magicPotions);
					uni.showToast({ title: `+${gain} ✨ 魔法药水`, icon: 'none' });
				}
				uni.setStorageSync('points', this.points);
			},
			draw(key) {
				const m = MACHINES[key];
				if (this.coins < m.cost) {
					uni.showToast({ title: '游戏币不足', icon: 'none' });
					return;
				}
				if (!m.pool || m.pool.length === 0) {
					uni.showToast({ title: '神秘大奖，敬请期待！', icon: 'none' });
					return;
				}
				this.coins -= m.cost;
				uni.setStorageSync('coins', this.coins);

				const pick = m.pool[Math.floor(Math.random() * m.pool.length)];
				const level = m.label.replace('扭蛋机', '');
				const newEgg = {
					id: Date.now(),
					name: '未知的蛋',
					level: level,
					source: level,
					animalName: pick.name,
					emoji: pick.emoji,
					progress: 0,
					// 图片系角色：带上 isSpecialImage + images
					isSpecialImage: !!pick.isSpecialImage,
					images: pick.images || null
				};
				const eggsList = uni.getStorageSync('eggsList') || [];
				eggsList.push(newEgg);
				uni.setStorageSync('eggsList', eggsList);
				// 如果当前没有激活的蛋，默认设为新蛋
				const activeId = uni.getStorageSync('activeEggId');
				if (!activeId) {
					uni.setStorageSync('activeEggId', newEgg.id);
				}

				this.resultLevel = newEgg.level;
				// 图片系特殊角色显示专属标题
				this.resultTitle2 = newEgg.isSpecialImage ? `魔性${pick.name}蛋` : '神秘的蛋';
				this.showResult = true;
			},
			closeResult() {
				this.showResult = false;
			}
		}
	};
</script>

<style scoped>
	/* 顶部三栏 */
	.top-bar {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: stretch;
		gap: 16rpx;
	}
	.stat-card {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4rpx;
		padding: 20rpx 0;
	}
	.stat-points { background: #FFD93D; }
	.stat-food { background: #FFB5D8; }
	.stat-coins { background: #4ECDC4; }
	.stat-emoji { font-size: 36rpx; }
	.stat-value { font-size: 44rpx; font-weight: 900; color: #333; }
	.stat-label { font-size: 22rpx; color: #5D4037; font-weight: 700; }

	/* 兑换所横幅 */
	.banner-btn {
		background: linear-gradient(90deg, #4ECDC4 0%, #80DEEA 100%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 6rpx;
		padding: 36rpx 30rpx;
	}
	.banner-text {
		font-size: 40rpx;
		font-weight: 900;
		color: #FFF;
		text-shadow: 4rpx 4rpx 0 #333;
	}
	.banner-sub {
		font-size: 22rpx;
		color: #FFF;
		font-weight: 700;
	}

	/* 扭蛋机区 */
	.machines-card { background: #FFF3C4; }
	.section-title {
		font-size: 36rpx;
		font-weight: 900;
		color: #333;
		margin-bottom: 24rpx;
		display: block;
	}
	.machines {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 24rpx;
	}
	.machine {
		border: 4rpx solid #333;
		border-radius: 24rpx;
		padding: 24rpx 16rpx 20rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12rpx;
		position: relative;
		box-shadow:
			8rpx 8rpx 0 #333,
			inset 0 6rpx 0 rgba(255, 255, 255, 0.55),
			inset 0 -8rpx 12rpx rgba(0, 0, 0, 0.18);
	}
	/* 四色机器主体渐变 */
	.machine-junior  { background: linear-gradient(180deg, #C8E6A0 0%, #95E065 60%, #6FB83F 100%); }
	.machine-middle  { background: linear-gradient(180deg, #80DEEA 0%, #4ECDC4 60%, #26A69A 100%); }
	.machine-senior  { background: linear-gradient(180deg, #D1B3FF 0%, #B197FC 60%, #7C4DFF 100%); }
	.machine-special { background: linear-gradient(135deg, #2C2C2C 0%, #FFD700 50%, #1A1A1A 100%); }

	/* 立体机身（玻璃球 + 金属底座） */
	.machine-body {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4rpx;
		margin-top: 6rpx;
	}
	.glass-dome {
		width: 130rpx;
		height: 130rpx;
		background:
			radial-gradient(circle at 30% 25%, rgba(255, 255, 255, 0.95) 0%, rgba(255, 255, 255, 0.35) 25%, rgba(255, 255, 255, 0.1) 60%, rgba(255, 255, 255, 0.05) 100%),
			radial-gradient(circle at 70% 75%, rgba(0, 0, 0, 0.08) 0%, rgba(0, 0, 0, 0) 60%);
		border: 4rpx solid #333;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
		box-shadow:
			inset 0 -10rpx 24rpx rgba(0, 0, 0, 0.18),
			inset 0 10rpx 16rpx rgba(255, 255, 255, 0.6),
			0 6rpx 0 rgba(0, 0, 0, 0.18);
	}
	.dome-content {
		font-size: 68rpx;
		text-shadow: 2rpx 2rpx 0 #FFF;
		position: relative;
		z-index: 1;
	}
	.dome-highlight {
		position: absolute;
		top: 14rpx;
		left: 22rpx;
		width: 38rpx;
		height: 26rpx;
		background: rgba(255, 255, 255, 0.9);
		border-radius: 50%;
		filter: blur(2rpx);
	}
	.dome-highlight-small {
		position: absolute;
		bottom: 22rpx;
		right: 30rpx;
		width: 14rpx;
		height: 10rpx;
		background: rgba(255, 255, 255, 0.6);
		border-radius: 50%;
		filter: blur(1rpx);
	}

	.metal-base {
		width: 92%;
		height: 36rpx;
		background: linear-gradient(180deg, #FFD700 0%, #B8860B 55%, #8B6914 100%);
		border: 4rpx solid #333;
		border-radius: 12rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		box-shadow:
			inset 0 2rpx 0 rgba(255, 255, 255, 0.6),
			inset 0 -4rpx 0 rgba(0, 0, 0, 0.3),
			0 4rpx 0 #333;
		margin-top: -4rpx;
	}
	.coin-slot {
		width: 50rpx;
		height: 10rpx;
		background: #333;
		border-radius: 5rpx;
		box-shadow: inset 0 2rpx 3rpx rgba(0, 0, 0, 0.6);
	}

	.machine-cost-tag {
		background: #FFD93D;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 4rpx 18rpx;
		box-shadow: 3rpx 3rpx 0 #333;
	}
	.cost-text {
		font-size: 22rpx;
		font-weight: 800;
		color: #333;
	}
	.draw-btn {
		width: 100%;
		background: #FF6B9D;
		border: 4rpx solid #333;
		border-radius: 14rpx;
		box-shadow: 4rpx 4rpx 0 #333;
		padding: 14rpx 0;
		display: flex;
		justify-content: center;
		align-items: center;
		transition: transform 0.1s, box-shadow 0.1s;
	}
	.draw-btn:active {
		transform: translate(4rpx, 4rpx);
		box-shadow: 0 0 0 #333;
	}
	.draw-text {
		color: #FFF;
		font-size: 26rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}
	.machine-label {
		font-size: 26rpx;
		font-weight: 900;
		color: #333;
	}
	.machine-hint {
		min-height: 36rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-wrap: wrap;
	}
	.hint-list {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
	}
	.hint-item {
		font-size: 20rpx;
		color: #333;
		font-weight: 700;
	}
	.hint-mystery {
		font-size: 22rpx;
		color: #FFD700;
		background: #333;
		padding: 4rpx 14rpx;
		border-radius: 10rpx;
		font-weight: 800;
		border: 2rpx solid #FFD700;
	}

	/* 通用 modal */
	.modal-mask {
		position: fixed;
		top: 0; left: 0; right: 0; bottom: 0;
		background: rgba(0, 0, 0, 0.55);
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.25s;
		z-index: 200;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.modal-mask.show {
		opacity: 1;
		pointer-events: auto;
	}

	.exchange-modal {
		background: #FFF9E6;
		width: 76%;
		padding: 50rpx 40rpx 40rpx;
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 16rpx;
		transform: scale(0.85);
		opacity: 0;
		transition: transform 0.25s, opacity 0.25s;
	}
	.modal-mask.show .exchange-modal,
	.modal-mask.show .result-modal {
		transform: scale(1);
		opacity: 1;
	}
	.close-btn {
		position: absolute;
		top: -20rpx;
		right: -20rpx;
		width: 60rpx;
		height: 60rpx;
		background: #FF5252;
		border: 4rpx solid #333;
		border-radius: 50%;
		box-shadow: 4rpx 4rpx 0 #333;
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 5;
	}
	.close-btn:active {
		transform: translate(4rpx, 4rpx);
		box-shadow: 0 0 0 #333;
	}
	.close-text {
		font-size: 40rpx;
		font-weight: 900;
		color: #FFF;
		line-height: 1;
	}
	.modal-title { font-size: 40rpx; font-weight: 900; color: #333; }
	.modal-sub { font-size: 24rpx; color: #5D4037; font-weight: 700; margin-bottom: 12rpx; }
	.exchange-btn { width: 100%; padding: 28rpx 0; }
	.food-exchange { background: #FFB5D8; }
	.coin-exchange { background: #4ECDC4; }
	.potion-exchange { background: #B197FC; }
	.magic-exchange { background: linear-gradient(90deg, #FFD700 0%, #FF6B9D 100%); }
	.food-exchange .btn-text,
	.coin-exchange .btn-text,
	.potion-exchange .btn-text,
	.magic-exchange .btn-text {
		color: #FFF;
		font-size: 32rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}

	/* 抽取结果弹窗 */
	.result-modal {
		background: linear-gradient(180deg, #FFF9E6 0%, #FFD93D 100%);
		width: 80%;
		padding: 50rpx 40rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 14rpx;
		transform: scale(0.85);
		opacity: 0;
		transition: transform 0.25s, opacity 0.25s;
		text-align: center;
	}
	.result-egg {
		font-size: 180rpx;
		line-height: 1;
		filter: drop-shadow(6rpx 6rpx 0 #333);
		animation: bounce 0.6s ease-out;
	}
	@keyframes bounce {
		0% { transform: scale(0); }
		60% { transform: scale(1.2); }
		100% { transform: scale(1); }
	}
	.result-title { font-size: 32rpx; font-weight: 800; color: #333; margin-top: 16rpx; }
	.result-level-tag {
		font-size: 36rpx;
		font-weight: 900;
		color: #FFF;
		background: #FF6B9D;
		padding: 6rpx 24rpx;
		border-radius: 14rpx;
		border: 3rpx solid #333;
		box-shadow: 4rpx 4rpx 0 #333;
	}
	.result-title2 { font-size: 32rpx; font-weight: 800; color: #333; }
	.result-tip { font-size: 24rpx; color: #5D4037; font-weight: 700; margin-bottom: 10rpx; }
	.confirm-btn { background: #95E065; padding: 22rpx 80rpx; margin-top: 10rpx; }
	.confirm-btn .btn-text { color: #333; font-size: 32rpx; font-weight: 800; }
</style>
