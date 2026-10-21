<template>
	<view class="mbe-page">
		<!-- 顶部信息卡 -->
		<view class="mbe-card info-card">
			<view class="info-row">
				<view class="info-cell">
					<text class="info-emoji">✨</text>
					<text class="info-num">{{ points }}</text>
					<text class="info-label">积分</text>
				</view>
				<view class="info-cell">
					<text class="info-emoji">🍖</text>
					<text class="info-num">{{ food }}</text>
					<text class="info-label">食物</text>
				</view>
				<view class="info-cell">
					<text class="info-emoji">🪙</text>
					<text class="info-num">{{ coins }}</text>
					<text class="info-label">游戏币</text>
				</view>
			</view>
			<view class="info-row info-row-2">
				<view class="info-cell">
					<text class="info-emoji">💊</text>
					<text class="info-num">{{ potions }}</text>
					<text class="info-label">药水</text>
				</view>
				<view class="info-cell">
					<text class="info-emoji">✨</text>
					<text class="info-num">{{ magicPotions }}</text>
					<text class="info-label">魔法药水</text>
				</view>
			</view>
		</view>

		<!-- 孵化区 -->
		<view class="mbe-card incubation-card" :class="eggAreaClass">
			<text class="section-title">🥚 孵化中</text>

			<view v-if="hasEgg" class="egg-area">
				<text class="egg-emoji" :class="eggGlowClass">{{ eggProgress >= 100 ? '✨🥚✨' : '🥚' }}</text>
				<text class="egg-progress-text">孵化进度：{{ eggProgress }}%</text>
				<view class="progress-bar">
					<view class="progress-fill" :style="{ width: eggProgress + '%' }"></view>
				</view>
				<view v-if="eggProgress >= 100" class="mbe-btn hatch-btn" @click="hatch">
					<text class="btn-text">✨ 破壳</text>
				</view>
				<text v-else class="egg-tip">{{ eggProgressTip }}</text>
			</view>

			<view v-else class="no-egg">
				<text class="no-egg-text">🥚 暂无蛋，去扭蛋页抽取吧～</text>
				<view class="mbe-btn goto-gacha-btn" @click="goGacha">
					<text class="btn-text">🎰 去扭蛋</text>
				</view>
			</view>
		</view>

		<!-- 场景区：我的家园 -->
		<view class="mbe-card scene-card">
			<text class="section-title">🏡 我的家园</text>
			<view class="scene">
				<view class="sky">
					<text class="sun">☀️</text>
					<text class="cloud cloud-1">☁️</text>
					<text class="cloud cloud-2">⛅</text>
				</view>
				<view class="ground">
					<text class="tree tree-left">🌲</text>
					<text class="flower flower-1">🌸</text>
					<text class="house">🏡</text>
					<text class="flower flower-2">🌻</text>
					<text class="tree tree-right">🌳</text>
				</view>
				<view class="fence">
					<view class="fence-piece" v-for="n in 9" :key="n"></view>
				</view>
			</view>
		</view>

		<!-- 动物之家列表 -->
		<view class="mbe-card animals-card">
			<text class="section-title">🐾 我的动物（{{ animals.length }}）</text>
			<view class="houses-grid">
				<view
					v-for="(a, i) in animals"
					:key="i"
					class="house-cell"
					:class="houseClass(a)"
					@click="openAnimal(i)"
				>
					<view class="house-visual">
						<text class="house-structure">{{ houseEmoji(a) }}</text>
						<text class="house-animal">{{ a.emoji }}</text>
						<text v-if="a.stage === '青年'" class="house-acc acc-scarf">🧣</text>
						<text v-else-if="a.stage === '成年'" class="house-acc acc-glasses">🕶️</text>
						<text v-else-if="a.stage === '老年'" class="house-acc acc-cane">🦯</text>
						<text v-if="a.status === 'sick'" class="status-icon">🤒</text>
						<text v-else-if="a.status === 'sleep'" class="status-icon">💤</text>
					</view>
					<text class="house-name">{{ a.name }}</text>
					<text class="house-stage-tag" :class="stageTagClass(a)">{{ a.stage }}</text>
				</view>
			</view>
			<view v-if="animals.length === 0" class="empty">
				<text class="empty-text">还没有动物～去孵化或扭蛋吧！</text>
			</view>
		</view>

		<!-- 动物详情弹窗 -->
		<view class="modal-mask" :class="{ show: showAnimal }" @click="closeAnimal">
			<view class="mbe-card animal-modal" @click.stop="">
				<view class="close-btn" @click="closeAnimal">
					<text class="close-text">×</text>
				</view>
				<view class="modal-emoji-wrap">
					<text class="modal-animal-emoji">{{ currentAnimal.emoji }}</text>
					<text v-if="currentAnimal.stage === '青年'" class="modal-acc acc-scarf">🧣</text>
					<text v-else-if="currentAnimal.stage === '成年'" class="modal-acc acc-glasses">🕶️</text>
					<text v-else-if="currentAnimal.stage === '老年'" class="modal-acc acc-cane">🦯</text>
				</view>
				<text class="modal-animal-name">{{ currentAnimal.name }}</text>
				<text class="modal-animal-stage">阶段：{{ currentAnimal.stage }}</text>
				<text class="modal-animal-status" :class="statusTextClass(currentAnimal.status)">
					状态：{{ statusText(currentAnimal.status) }}
				</text>
				<text class="growth-label">成长值 {{ currentAnimal.growth || 0 }}/{{ growthThreshold }}</text>
				<view class="progress-bar">
					<view class="progress-fill" :style="{ width: growthPercent + '%' }"></view>
				</view>
				<view v-if="currentAnimal.status === 'normal'" class="mbe-btn feed-btn" @click="feedCurrent">
					<text class="btn-text">🍖 喂养（消耗 1 食物）</text>
				</view>
				<view v-else-if="currentAnimal.status === 'sick'" class="mbe-btn heal-btn" @click="healCurrent">
					<text class="btn-text">💊 使用药水（治愈）</text>
				</view>
				<view v-else-if="currentAnimal.status === 'sleep'" class="mbe-btn wake-btn" @click="wakeCurrent">
					<text class="btn-text">✨ 使用魔法药水（唤醒）</text>
				</view>
			</view>
		</view>

		<!-- 破壳结果弹窗 -->
		<view class="modal-mask" :class="{ show: showHatchResult }" @click="closeHatchResult">
			<view class="mbe-card hatch-modal" @click.stop="">
				<text class="hatch-pet">{{ hatchResultEmoji }}</text>
				<text class="hatch-title">🎉 恭喜 {{ hatchResultEmoji }} {{ hatchResultName }}</text>
				<text class="hatch-title2">加入你的家园！</text>
				<view class="mbe-btn confirm-btn" @click="closeHatchResult">
					<text class="btn-text">确定</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const STAGES = ['幼年', '青年', '成年', '老年'];
	// 各阶段进阶所需成长值
	const STAGE_THRESHOLDS = {
		'幼年': 10,
		'青年': 20,
		'成年': 50,
		'老年': 50
	};
	const DAY_MS = 24 * 60 * 60 * 1000;

	export default {
		data() {
			return {
				points: 0,
				food: 0,
				coins: 0,
				potions: 0,
				magicPotions: 0,
				animals: [],
				currentEgg: null,
				showAnimal: false,
				currentAnimalIndex: -1,
				showHatchResult: false,
				hatchResultName: '',
				hatchResultEmoji: ''
			};
		},
		computed: {
			hasEgg() {
				return !!(this.currentEgg && this.currentEgg.name);
			},
			eggProgress() {
				return this.hasEgg ? (this.currentEgg.progress || 0) : 0;
			},
			eggProgressTip() {
				if (!this.hasEgg) return '';
				const map = { '初级': 10, '中级': 5, '高级': 3, '特级': 1 };
				const inc = map[this.currentEgg.level] || 10;
				return `答题答对 1 题 +${inc}% 进度`;
			},
			eggAreaClass() {
				if (!this.hasEgg) return '';
				const lv = this.currentEgg.level || '';
				if (lv === '高级') return 'egg-bg-senior';
				if (lv === '特级') return 'egg-bg-special';
				return '';
			},
			eggGlowClass() {
				if (!this.hasEgg) return '';
				const lv = this.currentEgg.level || '';
				if (lv === '高级') return 'glow-senior';
				if (lv === '特级') return 'glow-special';
				return '';
			},
			currentAnimal() {
				if (this.currentAnimalIndex >= 0 && this.currentAnimalIndex < this.animals.length) {
					return this.animals[this.currentAnimalIndex];
				}
				return { emoji: '', name: '', stage: '幼年', growth: 0, status: 'normal' };
			},
			growthThreshold() {
				return STAGE_THRESHOLDS[this.currentAnimal.stage] || 10;
			},
			growthPercent() {
				const g = this.currentAnimal.growth || 0;
				const t = this.growthThreshold;
				return Math.min(100, (g / t) * 100);
			}
		},
		onShow() {
			this.loadState();
			this.checkStatuses();
		},
		methods: {
			loadState() {
				this.points = uni.getStorageSync('points') || 0;
				this.food = uni.getStorageSync('food') || 0;
				this.coins = uni.getStorageSync('coins') || 0;
				this.potions = uni.getStorageSync('potions') || 0;
				this.magicPotions = uni.getStorageSync('magicPotions') || 0;
				const a = uni.getStorageSync('animals');
				this.animals = Array.isArray(a) ? a : [];
				const e = uni.getStorageSync('currentEgg');
				this.currentEgg = (e && typeof e === 'object' && e.name) ? e : null;
			},
			// 状态机：根据距离上次喂养的天数更新动物状态
			checkStatuses() {
				const now = Date.now();
				let changed = false;
				this.animals.forEach(a => {
					if (typeof a.lastFeedTime !== 'number') {
						a.lastFeedTime = now;
						a.status = 'normal';
						changed = true;
						return;
					}
					const diff = now - a.lastFeedTime;
					let newStatus = 'normal';
					if (diff >= 3 * DAY_MS) {
						newStatus = 'sleep';
					} else if (diff >= 1 * DAY_MS) {
						newStatus = 'sick';
					}
					if (a.status !== newStatus) {
						a.status = newStatus;
						changed = true;
					}
				});
				if (changed) {
					uni.setStorageSync('animals', this.animals);
					this.animals = this.animals.slice();
				}
			},
			// 房子类型：依据动物名映射
			houseClass(a) {
				const n = a.name;
				if (n === '小松鼠') return 'house-tree';
				if (n === '小鸡' || n === '小鸭') return 'house-cottage';
				if (n === '小猫' || n === '小狗') return 'house-box';
				if (/鱼|虾|蟹|龟|鲸|鲨|海/.test(n)) return 'house-pond';
				return 'house-cottage';
			},
			houseEmoji(a) {
				const c = this.houseClass(a);
				switch (c) {
					case 'house-tree': return '🌳';
					case 'house-cottage': return '🏡';
					case 'house-box': return '📦';
					case 'house-pond': return '🌊';
					default: return '🏡';
				}
			},
			stageTagClass(a) {
				if (a.status === 'sick') return 'tag-sick';
				if (a.status === 'sleep') return 'tag-sleep';
				return 'tag-normal';
			},
			statusText(s) {
				if (s === 'sick') return '生病';
				if (s === 'sleep') return '休眠';
				return '正常';
			},
			statusTextClass(s) {
				if (s === 'sick') return 'status-sick-text';
				if (s === 'sleep') return 'status-sleep-text';
				return 'status-normal-text';
			},
			goGacha() {
				uni.switchTab({ url: '/pages/points/points' });
			},
			// 破壳：把蛋里隐藏的 animalName/emoji 转成新动物
			hatch() {
				if (!this.currentEgg) return;
				let name = this.currentEgg.animalName;
				let em = this.currentEgg.emoji;
				if (!name || !em) {
					const fallback = [
						{ name: '小鸡', emoji: '🐤' },
						{ name: '小鸭', emoji: '🦆' },
						{ name: '小松鼠', emoji: '🐿️' }
					];
					const pick = fallback[Math.floor(Math.random() * fallback.length)];
					name = pick.name;
					em = pick.emoji;
				}
				const newAnimal = {
					name: name,
					emoji: em,
					stage: '幼年',
					growth: 0,
					lastFeedTime: Date.now(),
					status: 'normal'
				};
				this.animals.push(newAnimal);
				uni.setStorageSync('animals', this.animals);

				this.hatchResultName = name;
				this.hatchResultEmoji = em;
				this.showHatchResult = true;

				this.currentEgg = null;
				uni.setStorageSync('currentEgg', null);
			},
			closeHatchResult() {
				this.showHatchResult = false;
			},
			openAnimal(i) {
				this.currentAnimalIndex = i;
				this.showAnimal = true;
			},
			closeAnimal() {
				this.showAnimal = false;
			},
			// 喂养：消耗 1 食物 + 成长值 +1，更新 lastFeedTime，达到阈值进阶
			feedCurrent() {
				const idx = this.currentAnimalIndex;
				const animal = this.animals[idx];
				if (!animal) return;
				if (animal.status !== 'normal') {
					uni.showToast({ title: '动物当前状态无法喂养', icon: 'none' });
					return;
				}
				if (this.food <= 0) {
					uni.showToast({ title: '食物不足，快去兑换吧', icon: 'none' });
					return;
				}
				this.food -= 1;
				uni.setStorageSync('food', this.food);

				if (typeof animal.growth !== 'number') animal.growth = 0;
				animal.growth += 1;
				animal.lastFeedTime = Date.now();

				const threshold = STAGE_THRESHOLDS[animal.stage];
				if (threshold && animal.growth >= threshold) {
					const stageIdx = STAGES.indexOf(animal.stage);
					if (stageIdx >= 0 && stageIdx < STAGES.length - 1) {
						animal.stage = STAGES[stageIdx + 1];
						animal.growth = 0;
						uni.showToast({ title: `${animal.name} 进阶到 ${animal.stage}！`, icon: 'none' });
					} else {
						animal.growth = threshold;
						uni.showToast({ title: '已经达到最高阶段啦', icon: 'none' });
					}
				} else {
					uni.showToast({ title: '+1 成长值', icon: 'none' });
				}
				uni.setStorageSync('animals', this.animals);
				this.animals = this.animals.slice();
			},
			// 治愈生病动物：消耗 1 药水
			healCurrent() {
				const idx = this.currentAnimalIndex;
				const animal = this.animals[idx];
				if (!animal || animal.status !== 'sick') return;
				if (this.potions <= 0) {
					uni.showToast({ title: '药水不足，去兑换所买药水', icon: 'none' });
					return;
				}
				this.potions -= 1;
				uni.setStorageSync('potions', this.potions);
				animal.status = 'normal';
				animal.lastFeedTime = Date.now();
				uni.setStorageSync('animals', this.animals);
				uni.showToast({ title: `${animal.name} 已治愈！`, icon: 'none' });
				this.animals = this.animals.slice();
			},
			// 唤醒休眠动物：消耗 1 魔法药水
			wakeCurrent() {
				const idx = this.currentAnimalIndex;
				const animal = this.animals[idx];
				if (!animal || animal.status !== 'sleep') return;
				if (this.magicPotions <= 0) {
					uni.showToast({ title: '魔法药水不足，去兑换所买魔法药水', icon: 'none' });
					return;
				}
				this.magicPotions -= 1;
				uni.setStorageSync('magicPotions', this.magicPotions);
				animal.status = 'normal';
				animal.lastFeedTime = Date.now();
				uni.setStorageSync('animals', this.animals);
				uni.showToast({ title: `${animal.name} 已唤醒！`, icon: 'none' });
				this.animals = this.animals.slice();
			}
		}
	};
</script>

<style scoped>
	/* 顶部信息卡 */
	.info-card { background: #FFD93D; }
	.info-row {
		display: flex;
		justify-content: space-around;
		align-items: center;
	}
	.info-row-2 {
		margin-top: 16rpx;
		padding-top: 16rpx;
		border-top: 3rpx dashed #5D4037;
	}
	.info-cell {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 6rpx;
	}
	.info-emoji { font-size: 44rpx; }
	.info-num { font-size: 40rpx; font-weight: 800; color: #333; }
	.info-label { font-size: 22rpx; color: #5D4037; font-weight: 600; }

	/* 通用 section title */
	.section-title {
		font-size: 36rpx;
		font-weight: 900;
		color: #333;
		margin-bottom: 20rpx;
		display: block;
	}

	/* 孵化区 */
	.incubation-card {
		background: #4ECDC4;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 18rpx;
	}
	.egg-bg-senior { background: linear-gradient(135deg, #D1B3FF 0%, #B197FC 100%); }
	.egg-bg-special { background: linear-gradient(135deg, #2C2C2C 0%, #FFD700 50%, #1A1A1A 100%); }

	.egg-area {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 18rpx;
		width: 100%;
	}
	.egg-emoji {
		font-size: 180rpx;
		line-height: 1;
		filter: drop-shadow(6rpx 6rpx 0 #333);
	}
	.egg-emoji.glow-senior {
		filter: drop-shadow(6rpx 6rpx 0 #333) drop-shadow(0 0 12rpx #B197FC);
		animation: glow-pulse 1.5s ease-in-out infinite;
	}
	.egg-emoji.glow-special {
		filter: drop-shadow(6rpx 6rpx 0 #333) drop-shadow(0 0 18rpx #FFD700);
		animation: glow-pulse 1s ease-in-out infinite;
	}
	@keyframes glow-pulse {
		0%, 100% { transform: scale(1); }
		50% { transform: scale(1.05); }
	}
	.egg-progress-text {
		font-size: 28rpx;
		font-weight: 800;
		color: #333;
	}
	.progress-bar {
		width: 90%;
		height: 28rpx;
		background: #FFF;
		border: 4rpx solid #333;
		border-radius: 14rpx;
		overflow: hidden;
		box-shadow: 4rpx 4rpx 0 #333;
	}
	.progress-fill {
		height: 100%;
		background: linear-gradient(90deg, #FFB74D 0%, #FF7043 100%);
		transition: width 0.3s;
	}
	.hatch-btn {
		background: #FF6B9D;
		padding: 24rpx 80rpx;
		margin-top: 10rpx;
	}
	.hatch-btn .btn-text {
		color: #FFF;
		font-size: 36rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}
	.egg-tip {
		font-size: 22rpx;
		color: #333;
		font-weight: 600;
	}

	.no-egg {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 16rpx;
		padding: 20rpx;
	}
	.no-egg-text {
		font-size: 26rpx;
		color: #333;
		font-weight: 700;
	}
	.goto-gacha-btn {
		background: #FFD93D;
		padding: 20rpx 50rpx;
	}
	.goto-gacha-btn .btn-text {
		color: #333;
		font-size: 28rpx;
		font-weight: 800;
	}

	/* 场景区 */
	.scene-card {
		background: #FFF3C4;
	}
	.scene {
		position: relative;
		height: 320rpx;
		background: linear-gradient(180deg, #B3E5FC 0%, #B3E5FC 55%, #C5E1A5 55%, #C5E1A5 100%);
		border: 4rpx solid #333;
		border-radius: 20rpx;
		overflow: hidden;
		box-shadow: 6rpx 6rpx 0 #333;
	}
	.sun {
		position: absolute;
		top: 16rpx;
		right: 24rpx;
		font-size: 56rpx;
		filter: drop-shadow(2rpx 2rpx 0 #FFA500);
	}
	.cloud {
		position: absolute;
		font-size: 50rpx;
	}
	.cloud-1 { top: 30rpx; left: 30rpx; }
	.cloud-2 { top: 60rpx; left: 220rpx; }
	.tree {
		position: absolute;
		bottom: 40rpx;
		font-size: 80rpx;
		filter: drop-shadow(2rpx 2rpx 0 #5D4037);
	}
	.tree-left { left: 20rpx; }
	.tree-right { right: 20rpx; }
	.house {
		position: absolute;
		bottom: 40rpx;
		left: 50%;
		transform: translateX(-50%);
		font-size: 110rpx;
		filter: drop-shadow(3rpx 3rpx 0 #5D4037);
	}
	.flower {
		position: absolute;
		bottom: 30rpx;
		font-size: 36rpx;
	}
	.flower-1 { left: 140rpx; }
	.flower-2 { right: 140rpx; }
	.fence {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		display: flex;
		justify-content: space-around;
		padding: 0 10rpx;
		height: 32rpx;
		align-items: flex-end;
	}
	.fence-piece {
		width: 12rpx;
		height: 28rpx;
		background: #8B4513;
		border: 2rpx solid #333;
		border-radius: 4rpx 4rpx 0 0;
	}

	/* 动物之家 */
	.animals-card {
		background: #B5E85C;
	}
	.houses-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 20rpx;
	}
	.house-cell {
		border: 4rpx solid #333;
		border-radius: 24rpx;
		box-shadow: 6rpx 6rpx 0 #333;
		padding: 24rpx 16rpx 18rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 10rpx;
		transition: transform 0.1s;
	}
	.house-cell:active {
		transform: translate(4rpx, 4rpx);
		box-shadow: 2rpx 2rpx 0 #333;
	}
	/* 房子背景配色 */
	.house-tree {
		background: linear-gradient(180deg, #C5E1A5 0%, #66BB6A 100%);
	}
	.house-cottage {
		background: linear-gradient(180deg, #FFE0B2 0%, #FFCC80 100%);
	}
	.house-box {
		background: linear-gradient(180deg, #D7CCC8 0%, #BCAAA4 100%);
	}
	.house-pond {
		background: linear-gradient(180deg, #B3E5FC 0%, #4FC3F7 100%);
	}

	.house-visual {
		position: relative;
		width: 100%;
		height: 200rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.house-structure {
		font-size: 160rpx;
		position: absolute;
		z-index: 1;
		filter: drop-shadow(4rpx 4rpx 0 #333);
	}
	.house-animal {
		font-size: 72rpx;
		position: absolute;
		z-index: 2;
		bottom: 20rpx;
		filter: drop-shadow(2rpx 2rpx 0 #333);
	}
	.house-acc {
		position: absolute;
		font-size: 32rpx;
		z-index: 3;
		filter: drop-shadow(2rpx 2rpx 0 #333);
	}
	.acc-scarf {
		bottom: 30rpx;
		left: 50%;
		transform: translateX(-50%);
	}
	.acc-glasses {
		top: 60rpx;
		left: 50%;
		transform: translateX(-50%);
	}
	.acc-cane {
		bottom: 30rpx;
		right: 20rpx;
	}
	.status-icon {
		position: absolute;
		top: 10rpx;
		right: 16rpx;
		z-index: 4;
		font-size: 44rpx;
		animation: status-bob 1.2s ease-in-out infinite;
		filter: drop-shadow(2rpx 2rpx 0 #333);
	}
	@keyframes status-bob {
		0%, 100% { transform: translateY(0); }
		50% { transform: translateY(-6rpx); }
	}
	.house-name {
		font-size: 28rpx;
		font-weight: 800;
		color: #333;
	}
	.house-stage-tag {
		font-size: 22rpx;
		color: #FFF;
		padding: 4rpx 14rpx;
		border: 2rpx solid #333;
		border-radius: 12rpx;
		font-weight: 700;
	}
	.tag-normal { background: #FF6B9D; }
	.tag-sick { background: #FF5252; }
	.tag-sleep { background: #5C6BC0; }

	.empty {
		padding: 30rpx;
		text-align: center;
	}
	.empty-text {
		font-size: 26rpx;
		color: #555;
		font-weight: 600;
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

	/* 动物详情弹窗 */
	.animal-modal {
		background: #FFF9E6;
		width: 80%;
		padding: 50rpx 40rpx 40rpx;
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 14rpx;
		transform: scale(0.85);
		opacity: 0;
		transition: transform 0.25s, opacity 0.25s;
	}
	.modal-mask.show .animal-modal,
	.modal-mask.show .hatch-modal {
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
	.modal-emoji-wrap {
		position: relative;
		width: 160rpx;
		height: 160rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom: 10rpx;
	}
	.modal-animal-emoji {
		font-size: 130rpx;
		filter: drop-shadow(4rpx 4rpx 0 #333);
	}
	.modal-acc {
		position: absolute;
		font-size: 40rpx;
		z-index: 2;
		filter: drop-shadow(2rpx 2rpx 0 #333);
	}
	.modal-animal-name {
		font-size: 36rpx;
		font-weight: 900;
		color: #FF6B9D;
	}
	.modal-animal-stage {
		font-size: 26rpx;
		color: #5D4037;
		font-weight: 700;
	}
	.modal-animal-status {
		font-size: 28rpx;
		font-weight: 800;
		padding: 6rpx 18rpx;
		border: 3rpx solid #333;
		border-radius: 14rpx;
	}
	.status-normal-text { color: #333; background: #C5E1A5; }
	.status-sick-text { color: #FFF; background: #FF5252; }
	.status-sleep-text { color: #FFF; background: #5C6BC0; }

	.growth-label {
		font-size: 26rpx;
		color: #333;
		font-weight: 800;
		margin-top: 10rpx;
	}
	.feed-btn {
		background: #FFD93D;
		padding: 22rpx 50rpx;
		margin-top: 16rpx;
	}
	.feed-btn .btn-text {
		color: #333;
		font-size: 30rpx;
		font-weight: 800;
	}
	.heal-btn {
		background: #B197FC;
		padding: 22rpx 50rpx;
		margin-top: 16rpx;
	}
	.heal-btn .btn-text {
		color: #FFF;
		font-size: 30rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}
	.wake-btn {
		background: linear-gradient(90deg, #FFD700 0%, #FF6B9D 100%);
		padding: 22rpx 50rpx;
		margin-top: 16rpx;
	}
	.wake-btn .btn-text {
		color: #FFF;
		font-size: 30rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}

	/* 破壳结果弹窗 */
	.hatch-modal {
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
	.hatch-pet {
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
	.hatch-title {
		font-size: 32rpx;
		font-weight: 800;
		color: #333;
		margin-top: 10rpx;
	}
	.hatch-title2 {
		font-size: 32rpx;
		font-weight: 800;
		color: #333;
	}
	.confirm-btn {
		background: #95E065;
		padding: 22rpx 80rpx;
		margin-top: 14rpx;
	}
	.confirm-btn .btn-text {
		color: #333;
		font-size: 32rpx;
		font-weight: 800;
	}
</style>
