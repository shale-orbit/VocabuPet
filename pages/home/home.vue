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
		<view class="mbe-card incubation-card">
			<text class="section-title">🥚 孵化中（{{ eggsList.length }}）</text>

			<view v-if="eggsList.length > 0" class="eggs-list">
				<view
					v-for="(egg, index) in eggsList"
					:key="egg.id"
					class="egg-card"
					:class="[getEggAreaClass(egg), { 'egg-card-active': egg.id === activeEggId }]"
				>
					<view class="egg-emoji-wrap">
						<view class="egg-anim-container" :class="eggAnimClass(egg)">
							<text class="egg-emoji" :class="getEggGlowClass(egg)">{{ egg.progress >= 100 ? '✨🥚✨' : '🥚' }}</text>
						</view>
						<text v-if="egg.progress >= 70 && egg.progress < 100" class="egg-sparkle sp1">✨</text>
						<text v-if="egg.progress >= 70 && egg.progress < 100" class="egg-sparkle sp2">✨</text>
						<view v-if="index === particleIndex" class="particles">
							<text class="particle pt1">🎉</text>
							<text class="particle pt2">✨</text>
							<text class="particle pt3">🎉</text>
							<text class="particle pt4">✨</text>
						</view>
					</view>
					<text class="egg-source-text">{{ egg.source || egg.level || '初级' }}蛋</text>
					<text class="egg-progress-text">孵化进度：{{ egg.progress }}%</text>
					<view class="progress-bar">
						<view class="progress-fill" :style="{ width: egg.progress + '%' }"></view>
					</view>
					<!-- 激活且进度满：破壳 -->
					<view v-if="egg.id === activeEggId && egg.progress >= 100" class="mbe-btn hatch-btn" @click="hatch(index)">
						<text class="btn-text">✨ 破壳</text>
					</view>
					<!-- 激活但进度未满：优先孵化中 -->
					<view v-else-if="egg.id === activeEggId" class="mbe-btn active-hatch-btn">
						<text class="btn-text">⏳ 正在优先孵化中</text>
					</view>
					<!-- 未激活：设为优先 -->
					<view v-else class="mbe-btn set-active-btn" @click="setActiveEgg(egg.id)">
						<text class="btn-text">📌 设为优先孵化</text>
					</view>
				</view>
			</view>

			<view v-else class="no-egg">
				<text class="no-egg-text">🥚 当前没有正在孵化的蛋，快去扭蛋机抽一个吧！</text>
				<view class="mbe-btn goto-gacha-btn" @click="goGacha">
					<text class="btn-text">🎰 去扭蛋</text>
				</view>
			</view>
		</view>

		<!-- 动物之家列表 -->
		<view class="mbe-card animals-card">
			<text class="section-title">🐾 我的朋友（{{ animals.length }}）</text>
			<view class="houses-grid">
				<view
					v-for="(a, i) in animals"
					:key="i"
					class="house-cell"
					:class="houseClass(a)"
					@click="openAnimal(i)"
				>
					<view class="house-visual" :class="{ 'bounce-anim': i === bounceIndex }">
						<text class="house-structure">{{ houseEmoji(a) }}</text>
						<!-- 图片系角色：奶蛙 -->
						<image v-if="a.isSpecialImage" class="house-animal-image" :src="getAnimalImage(a)" mode="aspectFit" />
						<!-- Emoji 型动物：保留缩放 + 阶段配件 -->
						<template v-else>
							<text class="house-animal" :class="stageScaleClass(a)">{{ a.emoji }}</text>
							<text v-if="a.stage === '幼年'" class="house-acc acc-milk">🍼</text>
							<text v-else-if="a.stage === '青年'" class="house-acc acc-scarf">🧣</text>
							<text v-else-if="a.stage === '成年'" class="house-acc acc-glasses">🕶️</text>
							<text v-else-if="a.stage === '老年'" class="house-acc acc-cane">🦯</text>
						</template>
						<!-- 状态图标叠加：图片 / Emoji 都保留 -->
						<text v-if="a.status === 'sick'" class="status-icon">🤒</text>
						<text v-else-if="a.status === 'sleep'" class="status-icon">💤</text>
					</view>
					<text class="house-name">{{ a.nickname || a.name }}</text>
					<text class="house-source">来源：{{ a.source || '初级' }}</text>
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
					<image v-if="currentAnimal.isSpecialImage" class="modal-animal-image" :src="getAnimalImage(currentAnimal)" mode="aspectFit" />
					<template v-else>
						<text class="modal-animal-emoji" :class="stageScaleClass(currentAnimal)">{{ currentAnimal.emoji }}</text>
						<text v-if="currentAnimal.stage === '幼年'" class="modal-acc acc-milk">🍼</text>
						<text v-else-if="currentAnimal.stage === '青年'" class="modal-acc acc-scarf">🧣</text>
						<text v-else-if="currentAnimal.stage === '成年'" class="modal-acc acc-glasses">🕶️</text>
						<text v-else-if="currentAnimal.stage === '老年'" class="modal-acc acc-cane">🦯</text>
					</template>
					<text v-if="currentAnimal.status === 'sick'" class="status-icon">🤒</text>
					<text v-else-if="currentAnimal.status === 'sleep'" class="status-icon">💤</text>
				</view>
				<!-- 改名区 -->
				<view class="nickname-row">
					<text v-if="!editingNickname" class="modal-animal-name">{{ currentAnimal.nickname || currentAnimal.name }}</text>
					<text v-if="!editingNickname" class="edit-icon" @click="startEditNickname">✏️</text>
					<input v-if="editingNickname" class="nickname-input" v-model="nicknameInput" placeholder="输入新名字" />
					<view v-if="editingNickname" class="mbe-btn save-nick-btn" @click="saveNickname">
						<text class="btn-text">保存</text>
					</view>
				</view>
				<!-- 对话气泡 -->
				<view class="dialog-bubble">
					<text class="dialog-text">{{ getGreeting(currentAnimal) }}</text>
				</view>
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
				eggsList: [],
				activeEggId: null,
				particleIndex: -1,
				showAnimal: false,
				bounceIndex: -1,
				editingNickname: false,
				nicknameInput: '',
				currentAnimalIndex: -1,
				showHatchResult: false,
				hatchResultName: '',
				hatchResultEmoji: ''
			};
		},
		computed: {
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
				const eggs = uni.getStorageSync('eggsList');
				this.eggsList = Array.isArray(eggs) ? eggs : [];
				const activeId = uni.getStorageSync('activeEggId');
				this.activeEggId = activeId || null;
			},
			// 状态机：根据距离上次喂养的天数更新动物状态
			checkStatuses() {
				const now = Date.now();
				let changed = false;
				this.animals.forEach(a => {
					// 迁移 nickname 字段
					if (!a.nickname) {
						a.nickname = a.name;
						changed = true;
					}
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
			// 按阶段返回缩放类名
			stageScaleClass(a) {
				const s = a.stage;
				if (s === '幼年') return 'stage-young';
				if (s === '青年') return 'stage-teen';
				if (s === '成年') return 'stage-adult';
				if (s === '老年') return 'stage-old';
				return '';
			},
			// 图片系角色：按阶段返回对应图片地址
			getAnimalImage(a) {
				if (!a || !a.isSpecialImage || !a.images || a.images.length === 0) return '';
				const stageMap = { '幼年': 0, '青年': 1, '成年': 2, '老年': 3 };
				const idx = stageMap[a.stage];
				if (typeof idx !== 'number' || idx >= a.images.length) return a.images[0];
				return a.images[idx];
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
			// 蛋卡片样式（按品级）
			getEggAreaClass(egg) {
				if (!egg) return '';
				const lv = egg.level || '';
				if (lv === '高级') return 'egg-bg-senior';
				if (lv === '特级') return 'egg-bg-special';
				return '';
			},
			getEggGlowClass(egg) {
				if (!egg) return '';
				const lv = egg.level || '';
				if (lv === '高级') return 'glow-senior';
				if (lv === '特级') return 'glow-special';
				return '';
			},
			getEggTip(egg) {
				if (!egg) return '';
				const map = { '初级': 10, '中级': 5, '高级': 3, '特级': 1 };
				const inc = map[egg.level] || 10;
				return `答题答对 1 题 +${inc}% 进度`;
			},
			// 根据进度返回蛋的动画状态类
			eggAnimClass(egg) {
				const p = egg.progress || 0;
				if (p >= 100) return 'egg-pulse';
				if (p >= 70) return 'egg-shake';
				if (p >= 30) return 'egg-wiggle';
				return '';
			},
			// 设为优先孵化
			setActiveEgg(eggId) {
				this.activeEggId = eggId;
				uni.setStorageSync('activeEggId', eggId);
			},
			// 破壳：粒子特效 → 600ms 后执行实际破壳
			hatch(index) {
				const egg = this.eggsList[index];
				if (!egg) return;
				// 触发粒子特效
				this.particleIndex = index;
				setTimeout(() => {
					let name = egg.animalName;
					let em = egg.emoji;
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
						status: 'normal',
						source: (egg && egg.source) || '初级',
						// 图片系角色：蛋里的 images + isSpecialImage 一并流转给动物
						isSpecialImage: !!(egg && egg.isSpecialImage),
						images: (egg && egg.images) ? egg.images.slice() : null
					};
					this.animals.push(newAnimal);
					uni.setStorageSync('animals', this.animals);

					this.hatchResultName = name;
					this.hatchResultEmoji = em;
					this.showHatchResult = true;

					// 从孵化队列中移除该蛋
					this.eggsList.splice(index, 1);
					uni.setStorageSync('eggsList', this.eggsList);
					this.particleIndex = -1;
					// 清空激活蛋，等待用户下次选择
					this.activeEggId = null;
					uni.setStorageSync('activeEggId', null);
				}, 600);
			},
			closeHatchResult() {
				this.showHatchResult = false;
			},
			openAnimal(i) {
				// 先弹跳动画，300ms 后再打开面板
				this.bounceIndex = i;
				setTimeout(() => {
					this.currentAnimalIndex = i;
					this.showAnimal = true;
					this.bounceIndex = -1;
					this.editingNickname = false;
				}, 300);
			},
			closeAnimal() {
				this.showAnimal = false;
				this.editingNickname = false;
			},
			// 智能体对话：时间问候 + 状态反馈 + 随机鼓励
			getGreeting(animal) {
				if (!animal || !animal.name) return '';
				const h = new Date().getHours();
				let timeGreeting;
				if (h >= 6 && h < 12) timeGreeting = '早上好呀';
				else if (h >= 12 && h < 18) timeGreeting = '下午好呀';
				else if (h >= 18 && h < 24) timeGreeting = '晚上好呀';
				else timeGreeting = '夜深了';
				const nick = animal.nickname || animal.name;
				let msg;
				if (animal.status === 'sick') {
					msg = '我好像生病了，主人能给我用药水吗？';
				} else if (animal.status === 'sleep') {
					msg = 'Zzz... 我还在睡觉哦...';
				} else if (animal.name === '奶蛙') {
					// 奶蛙专属魔性语录
					const naibaTips = [
						'今天你背单词了吗？没背我可要生气了哦！',
						'你看我这清澈的眼神，像不像你还没记住的abandon？',
						'别摸鱼了，快带我去背单词！',
						'只要你背单词，我就是你最忠实的胖胖！',
						'我这大肚子里面装的都是知识，你信吗？'
					];
					msg = naibaTips[Math.floor(Math.random() * naibaTips.length)];
				} else {
					const tips = ['今天背单词了吗？', '坚持就是胜利！', '主人，加油哦！', '一起变强吧！'];
					msg = tips[Math.floor(Math.random() * tips.length)];
				}
				return `${timeGreeting}，${nick}！${msg}`;
			},
			// 开始改昵称
			startEditNickname() {
				this.nicknameInput = this.currentAnimal.nickname || this.currentAnimal.name;
				this.editingNickname = true;
			},
			// 保存昵称
			saveNickname() {
				const idx = this.currentAnimalIndex;
				if (idx < 0 || !this.animals[idx]) return;
				this.animals[idx].nickname = this.nicknameInput || this.animals[idx].name;
				uni.setStorageSync('animals', this.animals);
				this.editingNickname = false;
				this.animals = this.animals.slice();
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

	.eggs-list {
		display: flex;
		flex-direction: column;
		gap: 24rpx;
		width: 100%;
	}
	.egg-card {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 18rpx;
		width: 100%;
		padding: 28rpx 20rpx;
		border: 4rpx solid #333;
		border-radius: 20rpx;
		background: #FFF9E6;
		box-shadow: 6rpx 6rpx 0 #333;
	}
	/* 激活态高亮 */
	.egg-card-active {
		border: 6rpx solid #FF6B9D !important;
		box-shadow: 6rpx 6rpx 0 #333, 0 0 20rpx rgba(255, 107, 157, 0.4) !important;
		background: #FFF9E6 !important;
	}
	/* 优先孵化中按钮（不可点） */
	.active-hatch-btn {
		background: #E0E0E0;
		pointer-events: none;
	}
	.active-hatch-btn .btn-text {
		color: #666;
		font-size: 28rpx;
		font-weight: 800;
	}
	/* 设为优先孵化按钮 */
	.set-active-btn {
		background: #80DEEA;
	}
	.set-active-btn .btn-text {
		color: #333;
		font-size: 28rpx;
		font-weight: 800;
	}
	/* 蛋动画容器 */
	.egg-emoji-wrap {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		min-height: 200rpx;
	}
	.egg-anim-container {
		display: inline-flex;
		justify-content: center;
		align-items: center;
	}
	/* 苏醒期：缓慢晃动 */
	.egg-wiggle {
		animation: wiggle 2s ease-in-out infinite;
	}
	@keyframes wiggle {
		0%, 100% { transform: rotate(-5deg); }
		50% { transform: rotate(5deg); }
	}
	/* 躁动期：剧烈晃动 + 红色发光 */
	.egg-shake {
		animation: shake 0.3s ease-in-out infinite;
	}
	.egg-shake .egg-emoji {
		filter: drop-shadow(6rpx 6rpx 0 #333) drop-shadow(0 0 12rpx rgba(255, 50, 50, 0.6));
	}
	@keyframes shake {
		0%, 100% { transform: translate(0, 0) rotate(0deg); }
		25% { transform: translate(-4rpx, -2rpx) rotate(-3deg); }
		50% { transform: translate(4rpx, 2rpx) rotate(3deg); }
		75% { transform: translate(-2rpx, 4rpx) rotate(-2deg); }
	}
	/* 待破壳：发光闪烁 */
	.egg-pulse {
		animation: pulse 1s ease-in-out infinite;
	}
	@keyframes pulse {
		0%, 100% { transform: scale(1); filter: drop-shadow(0 0 0 transparent); }
		50% { transform: scale(1.08); filter: drop-shadow(0 0 16rpx rgba(255, 215, 0, 0.8)); }
	}
	/* 躁动期闪烁星星 */
	.egg-sparkle {
		position: absolute;
		font-size: 32rpx;
		animation: sparkle 1s ease-in-out infinite;
		pointer-events: none;
	}
	.sp1 { top: 10rpx; right: 30%; }
	.sp2 { top: 20rpx; left: 25%; animation-delay: 0.5s; }
	@keyframes sparkle {
		0%, 100% { opacity: 0; transform: scale(0.5); }
		50% { opacity: 1; transform: scale(1.2); }
	}
	/* 破壳粒子特效 */
	.particles {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		pointer-events: none;
	}
	.particle {
		position: absolute;
		font-size: 40rpx;
		animation: particle-float 0.6s ease-out forwards;
	}
	.pt1 { left: -60rpx; animation-delay: 0s; }
	.pt2 { left: -20rpx; animation-delay: 0.1s; }
	.pt3 { left: 20rpx; animation-delay: 0.05s; }
	.pt4 { left: 60rpx; animation-delay: 0.15s; }
	@keyframes particle-float {
		0% { transform: translateY(0) scale(0.5); opacity: 1; }
		100% { transform: translateY(-200rpx) scale(1.5); opacity: 0; }
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
	.egg-source-text {
		font-size: 26rpx;
		font-weight: 800;
		color: #FF6B9D;
		background: #FFF;
		padding: 4rpx 16rpx;
		border: 2rpx solid #333;
		border-radius: 10rpx;
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

	/* 动物之家 */
	.animals-card {
		background: #B5E85C;
	}
	.houses-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 28rpx;
	}
	.house-cell {
		border: 4rpx solid #333;
		border-radius: 24rpx;
		box-shadow: 6rpx 6rpx 0 #333;
		padding: 32rpx 20rpx 24rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12rpx;
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
		height: 260rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.house-structure {
		font-size: 190rpx;
		position: absolute;
		z-index: 1;
		filter: drop-shadow(4rpx 4rpx 0 #333);
	}
		.house-animal {
			font-size: 88rpx;
			position: absolute;
			z-index: 2;
			bottom: 24rpx;
			filter: drop-shadow(2rpx 2rpx 0 #333);
		}
		/* 图片系动物（奶蛙）：固定 160rpx，object-fit: contain */
		.house-animal-image {
			width: 160rpx;
			height: 160rpx;
			position: absolute;
			z-index: 2;
			bottom: 24rpx;
			object-fit: contain;
			filter: drop-shadow(2rpx 2rpx 0 #333);
		}
	/* 阶段缩放 */
	.stage-young .house-animal { transform: scale(0.7); }
	.stage-teen .house-animal { transform: scale(1.0); }
	.stage-adult .house-animal { transform: scale(1.2); filter: drop-shadow(2rpx 2rpx 0 #333) drop-shadow(0 0 8rpx rgba(255, 215, 0, 0.6)); }
	.stage-old .house-animal { transform: scale(1.1); filter: drop-shadow(2rpx 2rpx 0 #333) grayscale(0.5); }
	/* 弹跳动画 */
	.bounce-anim { animation: bounce-quick 0.3s ease-out; }
	@keyframes bounce-quick {
		0% { transform: scale(1); }
		50% { transform: scale(1.3); }
		100% { transform: scale(1); }
	}
	/* 奶嘴配件 */
	.acc-milk {
		bottom: 50rpx;
		left: 50%;
		transform: translateX(-50%);
		font-size: 36rpx;
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
	.house-source {
		font-size: 20rpx;
		color: #5D4037;
		font-weight: 600;
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
		/* 图片系动物模态框：固定 160rpx */
		.modal-animal-image {
			width: 160rpx;
			height: 160rpx;
			object-fit: contain;
			filter: drop-shadow(4rpx 4rpx 0 #333);
		}
	/* 模态框阶段缩放 */
	.stage-young .modal-animal-emoji { transform: scale(0.7); }
	.stage-teen .modal-animal-emoji { transform: scale(1.0); }
	.stage-adult .modal-animal-emoji { transform: scale(1.2); filter: drop-shadow(4rpx 4rpx 0 #333) drop-shadow(0 0 10rpx rgba(255, 215, 0, 0.6)); }
	.stage-old .modal-animal-emoji { transform: scale(1.1); filter: drop-shadow(4rpx 4rpx 0 #333) grayscale(0.5); }
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
	/* 改名区 */
	.nickname-row {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 12rpx;
		margin: 8rpx 0;
	}
	.edit-icon { font-size: 32rpx; }
	.nickname-input {
		border: 4rpx solid #333;
		border-radius: 12rpx;
		padding: 10rpx 20rpx;
		font-size: 30rpx;
		width: 300rpx;
		background: #FFF;
	}
	.save-nick-btn { background: #95E065; padding: 10rpx 30rpx; }
	.save-nick-btn .btn-text { color: #333; font-size: 26rpx; font-weight: 800; }
	/* 对话气泡 */
	.dialog-bubble {
		background: #FFF9E6;
		border: 4rpx solid #333;
		border-radius: 20rpx;
		padding: 20rpx 24rpx;
		margin: 16rpx 0;
		box-shadow: 4rpx 4rpx 0 #333;
		position: relative;
	}
	.dialog-bubble::before {
		content: '';
		position: absolute;
		top: -16rpx;
		left: 50%;
		transform: translateX(-50%);
		width: 0;
		height: 0;
		border-left: 16rpx solid transparent;
		border-right: 16rpx solid transparent;
		border-bottom: 16rpx solid #333;
	}
	.dialog-text {
		font-size: 28rpx;
		color: #333;
		font-weight: 700;
		line-height: 1.5;
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
