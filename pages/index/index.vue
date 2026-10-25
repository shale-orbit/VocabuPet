<template>
	<view class="mbe-page">
		<!-- 顶部：左积分卡 + 中复习按钮 + 右书本按钮 -->
		<view class="top-bar">
			<view class="mbe-card points-card">
				<text class="points-text">✨ 今日积分：{{ points }}</text>
			</view>
			<view class="mbe-btn review-btn" @click="openReview">
				<text class="book-icon">📚</text>
				<text v-if="wrongCount > 0" class="red-dot">{{ wrongCount }}</text>
			</view>
			<view class="mbe-btn book-btn" @click="openVocab">
				<text class="book-icon">📖</text>
			</view>
		</view>
		<text class="library-text">📚 当前词库：{{ currentLibraryName }}</text>

		<!-- 模式切换开关 -->
		<view class="mode-switch">
			<view
				class="mbe-btn mode-btn"
				:class="{ 'active-en': quizMode === 'en2zh' }"
				@click="switchMode('en2zh')"
			>
				<text class="mode-text">英 → 中</text>
			</view>
			<view
				class="mbe-btn mode-btn"
				:class="{ 'active-zh': quizMode === 'zh2en' }"
				@click="switchMode('zh2en')"
			>
				<text class="mode-text">中 → 英</text>
			</view>
		</view>

		<!-- 题干卡片 -->
		<view class="mbe-card question-card">
			<text class="question-label">{{ quizMode === 'en2zh' ? '选出正确的中文释义' : '选出正确的英文单词' }}</text>
			<text class="question-text">{{ questionText }}</text>
		</view>

		<!-- 选项按钮（竖向 3 个） -->
		<view class="options">
			<view
				v-for="(opt, i) in shuffledOptions"
				:key="i"
				class="mbe-btn option-btn"
				:class="getOptionClass(i)"
				@click="chooseOption(i)"
			>
				<text class="option-text">{{ opt }}</text>
			</view>
		</view>

		<!-- 答题详情气泡（无论对错都显示） -->
		<view v-if="answered" class="mbe-card detail-bubble">
			<!-- 单词信息行：word [phonetic] partOfSpeech -->
			<view class="word-info-row">
				<text class="detail-word">{{ current.word }}</text>
				<text class="detail-phonetic">[{{ current.phonetic }}]</text>
				<text class="detail-pos">{{ current.partOfSpeech }}</text>
			</view>
			<text class="detail-translation">✅ 正确释义：{{ current.translation }}</text>

			<!-- 发音按钮 -->
			<view class="audio-row">
				<view class="mbe-btn audio-btn" @click="playAudio(1)">
					<text class="btn-text">🔊 英音</text>
				</view>
				<view class="mbe-btn audio-btn" @click="playAudio(2)">
					<text class="btn-text">🔊 美音</text>
				</view>
			</view>

			<!-- 例句 -->
			<view class="example-section">
				<text class="example-en">📝 {{ current.example }}</text>
				<text class="example-zh">📖 {{ current.exampleZh }}</text>
			</view>

			<!-- 扩充知识 -->
			<view class="extension-card">
				<text class="extension-text">💡 {{ current.extension }}</text>
			</view>
		</view>

		<!-- 上一个 / 下一个 导航 -->
		<view class="nav-row">
			<view
				class="mbe-btn nav-btn prev-btn"
				:class="{ 'disabled': currentIndex === 0 }"
				@click="goPrev"
			>
				<text class="btn-text">⬅️ 上一个</text>
			</view>
			<view
				class="mbe-btn nav-btn next-btn"
				:class="{ 'disabled': !answered }"
				@click="goNext"
			>
				<text class="btn-text">下一个 ➡️</text>
			</view>
		</view>

		<!-- 词汇管理弹窗：遮罩 -->
		<view
			class="vocab-mask"
			:class="{ show: showVocab }"
			@click="closeVocab"
		></view>

		<!-- 词汇管理弹窗：底部抽屉 -->
		<view class="vocab-sheet" :class="{ show: showVocab }" @click.stop="">
			<view class="sheet-handle"></view>
			<view class="close-btn" @click="closeVocab">
				<text class="close-text">×</text>
			</view>

			<!-- 查字典 -->
			<view class="vocab-section">
				<text class="section-title">📖 查字典</text>
				<view class="search-row">
					<input
						class="mbe-input"
						v-model="searchInput"
						placeholder="输入英文单词"
						placeholder-class="ph"
						confirm-type="search"
						@confirm="doSearch"
					/>
					<view class="mbe-btn search-btn" @click="doSearch">
						<text class="btn-text">搜索</text>
					</view>
				</view>
				<view v-if="searchResult" class="mbe-card search-result-card">
					<view v-if="!searchResult.notFound" class="result-box">
						<text class="result-word">{{ searchResult.word }}</text>
						<text v-if="searchResult.phonetic" class="result-phonetic">[{{ searchResult.phonetic }}]</text>
						<text class="result-trans">{{ searchResult.translation }}</text>
						<text v-if="searchResult.example" class="result-example">📝 {{ searchResult.example }}</text>
						<text v-if="searchResult.exampleZh" class="result-example-zh">📖 {{ searchResult.exampleZh }}</text>
						<text class="result-source">📚 来源：{{ searchResult.source }}</text>
					</view>
					<text v-else class="not-found">😮 词库暂无</text>
				</view>
			</view>

			<!-- 可选词库 -->
			<view class="vocab-section">
				<text class="section-title">📚 可选词库</text>
				<view class="library-buttons">
					<view
						class="lib-btn"
						:class="{ 'lib-active': currentLibrary === 'CET-4' }"
						@click="switchLibrary('CET-4')"
					>
						<text>CET-4 核心词</text>
					</view>
					<view
						class="lib-btn"
						:class="{ 'lib-active': currentLibrary === 'CET-6' }"
						@click="switchLibrary('CET-6')"
					>
						<text>CET-6 核心词</text>
					</view>
					<view
						class="lib-btn"
						:class="{ 'lib-active': currentLibrary === 'IELTS' }"
						@click="switchLibrary('IELTS')"
					>
						<text>雅思核心词</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import cet4Data from '@/static/cet4.json';
	import cet6Data from '@/static/cet6.json';
	import ieltsData from '@/static/ielts.json';

	const LIBRARY_DATA = {
		'CET-4': cet4Data,
		'CET-6': cet6Data,
		'IELTS': ieltsData
	};

	export default {
		data() {
			return {
				words: cet4Data,
				points: 0,
				streak: 1,
				currentLibrary: 'CET-4',
				libraries: [
					{ name: 'CET-4 核心词', key: 'CET-4' },
					{ name: 'CET-6 核心词', key: 'CET-6' },
					{ name: '雅思核心词', key: 'IELTS' }
				],
				quizMode: 'en2zh',
				currentIndex: 0,
				shuffledOptions: [],
				correctIndex: 0,
				selected: -1,
				answered: false,
				isCorrect: false,
				showVocab: false,
				searchInput: '',
				searchResult: null,
				libraries: ['CET-4 核心词', '考研高频词', '校园生活词汇'],
				myWords: [],
				wrongCount: 0,
				audioCtx: null
			};
		},
		computed: {
			current() {
				return this.words[this.currentIndex] || this.words[0];
			},
			questionText() {
				if (!this.current || !this.current.word) return '';
				return this.quizMode === 'en2zh' ? this.current.word : this.current.translation;
			},
			currentLibraryName() {
				const lib = this.libraries.find(l => l.key === this.currentLibrary);
				return lib ? lib.name : 'CET-4 核心词';
			}
		},
		onLoad() {
			this.loadState();
			this.regenerateOptions();
		},
		onShow() {
			this.loadState();
			if (!this.answered) {
				this.regenerateOptions();
			}
		},
		onUnload() {
			this.destroyAudio();
		},
		onHide() {
			this.destroyAudio();
		},
		methods: {
			loadState() {
				this.points = uni.getStorageSync('points') || 0;
				this.streak = uni.getStorageSync('streak') || 1;
				// 读取词库设置，迁移旧值
				const stored = uni.getStorageSync('currentLibrary');
				if (stored === 'IELTS' || stored === '雅思核心词') {
					this.currentLibrary = 'IELTS';
				} else if (stored === 'CET-6' || stored === 'CET-6 核心词') {
					this.currentLibrary = 'CET-6';
				} else {
					this.currentLibrary = 'CET-4';
				}
				// 动态加载对应词库数据
				this.words = LIBRARY_DATA[this.currentLibrary] || LIBRARY_DATA['CET-4'];
				const mw = uni.getStorageSync('myWords');
				this.myWords = Array.isArray(mw) ? mw : [];
				const ww = uni.getStorageSync('wrongWords');
				this.wrongCount = Array.isArray(ww) ? ww.length : 0;
			},
			regenerateOptions() {
				if (!this.current || !this.current.word) {
					this.shuffledOptions = [];
					this.correctIndex = 0;
					return;
				}
				let correct, distractors;
				if (this.quizMode === 'en2zh') {
					correct = this.current.translation;
					distractors = (this.current.distractorsZh || []).slice();
				} else {
					correct = this.current.word;
					distractors = (this.current.distractorsEn || []).slice();
				}
				const all = [correct, ...distractors];
				this.shuffledOptions = this.shuffle(all);
				this.correctIndex = this.shuffledOptions.indexOf(correct);
			},
			shuffle(arr) {
				const a = arr.slice();
				for (let i = a.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[a[i], a[j]] = [a[j], a[i]];
				}
				return a;
			},
			switchMode(mode) {
				if (this.quizMode === mode) return;
				this.quizMode = mode;
				this.resetAnswer();
				this.regenerateOptions();
			},
			resetAnswer() {
				this.selected = -1;
				this.answered = false;
				this.isCorrect = false;
			},
			getOptionClass(i) {
				if (!this.answered) return '';
				if (i === this.selected && i === this.correctIndex) return 'correct';
				if (i === this.selected && i !== this.correctIndex) return 'wrong';
				if (i === this.correctIndex) return 'correct';
				return 'dim';
			},
			chooseOption(i) {
				if (this.answered) return;
				this.selected = i;
				this.answered = true;
				this.isCorrect = (i === this.correctIndex);

				if (this.isCorrect) {
					this.points += 1;
					uni.setStorageSync('points', this.points);

					// 只有被激活的蛋才增加进度
					const eggs = uni.getStorageSync('eggsList');
					const activeId = uni.getStorageSync('activeEggId');
					if (Array.isArray(eggs) && eggs.length > 0 && activeId) {
						const incMap = { '初级': 10, '中级': 5, '高级': 3, '特级': 1 };
						const egg = eggs.find(e => e.id === activeId);
						if (egg && egg.progress < 100) {
							const inc = incMap[egg.level] || 10;
							egg.progress = Math.min(100, (egg.progress || 0) + inc);
							uni.setStorageSync('eggsList', eggs);
						}
					}
					// 已取消自动跳转，等待用户手动点击「下一个」
				} else {
					// 答错：把当前单词完整对象存入 wrongWords（去重）
					if (this.current && this.current.word) {
						const ww = uni.getStorageSync('wrongWords');
						const arr = Array.isArray(ww) ? ww : [];
						if (!arr.some(w => w.word === this.current.word)) {
							arr.push({ ...this.current });
							uni.setStorageSync('wrongWords', arr);
							this.wrongCount = arr.length;
						}
					}
				}
				// 答错同样停留，等待用户手动操作
			},
			// 跳转到复习页
			openReview() {
				uni.navigateTo({ url: '/pages/review/review' });
			},
			// 上一个：回到上一题并重置作答状态
			goPrev() {
				if (this.currentIndex === 0) return;
				this.destroyAudio();
				this.currentIndex -= 1;
				this.resetAnswer();
				this.regenerateOptions();
			},
			// 下一个：必须已作答才能进入下一题，防止疯狂乱点
			goNext() {
				if (!this.answered) {
					uni.showToast({ title: '请先选择答案并查看解析', icon: 'none' });
					return;
				}
				if (this.currentIndex >= this.words.length - 1) {
					uni.showToast({ title: '已经是最后一题啦', icon: 'none' });
					return;
				}
				this.destroyAudio();
				this.currentIndex += 1;
				this.resetAnswer();
				this.regenerateOptions();
			},
			// 语音播放（有道网络直链）
			playAudio(type) {
				this.destroyAudio();
				if (!this.current || !this.current.word) return;
				const ctx = uni.createInnerAudioContext();
				ctx.src = `https://dict.youdao.com/dictvoice?audio=${encodeURIComponent(this.current.word)}&type=${type}`;
				this.audioCtx = ctx;
				try { ctx.play(); } catch (e) {}
			},
			destroyAudio() {
				if (this.audioCtx) {
					try { this.audioCtx.destroy(); } catch (e) {}
					this.audioCtx = null;
				}
			},
			// 词汇管理弹窗
			openVocab() {
				this.showVocab = true;
			},
			closeVocab() {
				this.showVocab = false;
			},
			doSearch() {
				const q = (this.searchInput || '').trim().toLowerCase();
				if (!q) {
					this.searchResult = null;
					return;
				}
				const foundInCet = this.words.find(w => w.word.toLowerCase() === q);
				if (foundInCet) {
					this.searchResult = { ...foundInCet, source: 'CET-4 核心词', notFound: false };
					return;
				}
				const foundInMine = this.myWords.find(w => w.word.toLowerCase() === q);
				if (foundInMine) {
					this.searchResult = { ...foundInMine, source: '我的词库', notFound: false };
					return;
				}
				this.searchResult = { notFound: true };
			},
			// 切换词库：动态加载对应题库，重置进度与答题状态
			switchLibrary(libKey) {
				if (this.currentLibrary === libKey) return;
				this.currentLibrary = libKey;
				uni.setStorageSync('currentLibrary', libKey);
				this.words = LIBRARY_DATA[libKey];
				this.currentIndex = 0;
				this.destroyAudio();
				this.resetAnswer();
				this.regenerateOptions();
				this.showVocab = false;
				const lib = this.libraries.find(l => l.key === libKey);
				const toastMsg = lib ? `已切换至${lib.name}` : '已切换词库';
				uni.showToast({ title: toastMsg, icon: 'none' });
			}
		}
	};
</script>

<style scoped>
	/* 顶部 */
	.top-bar {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		gap: 20rpx;
	}
	.points-card {
		background: #FFD93D;
		flex: 1;
	}
	.points-text {
		font-size: 36rpx;
		font-weight: 800;
		color: #333;
	}
	.book-btn {
		background: #4ECDC4;
		padding: 18rpx 30rpx;
	}
	.review-btn {
		background: #FFB5D8;
		padding: 18rpx 30rpx;
		position: relative;
	}
	.red-dot {
		position: absolute;
		top: -10rpx;
		right: -10rpx;
		min-width: 36rpx;
		height: 36rpx;
		padding: 0 8rpx;
		background: #FF5252;
		color: #FFF;
		font-size: 22rpx;
		font-weight: 900;
		border: 3rpx solid #333;
		border-radius: 18rpx;
		box-shadow: 3rpx 3rpx 0 #333;
		display: flex;
		justify-content: center;
		align-items: center;
		line-height: 1;
	}
	.book-icon {
		font-size: 44rpx;
	}
	.library-text {
		font-size: 24rpx;
		color: #555;
		font-weight: 700;
		text-align: center;
		margin-top: -8rpx;
	}

	/* 模式开关 */
	.mode-switch {
		display: flex;
		gap: 16rpx;
	}
	.mode-btn {
		flex: 1;
		background: #FFFFFF;
		padding: 18rpx 0;
	}
	.mode-text {
		font-size: 30rpx;
		font-weight: 800;
		color: #333;
	}
	.mode-btn.active-en { background: #FF6B9D; }
	.mode-btn.active-en .mode-text { color: #FFF; }
	.mode-btn.active-zh { background: #B197FC; }
	.mode-btn.active-zh .mode-text { color: #FFF; }

	/* 题干 */
	.question-card {
		background: #FFD93D;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 18rpx;
		padding: 50rpx 30rpx;
	}
	.question-label {
		font-size: 24rpx;
		color: #5D4037;
		font-weight: 700;
		background: #FFF;
		padding: 6rpx 20rpx;
		border-radius: 14rpx;
		border: 3rpx solid #333;
	}
	.question-text {
		font-size: 88rpx;
		font-weight: 900;
		color: #333;
		letter-spacing: 2rpx;
		/* 移除 3D text-shadow，文字平实护眼 */
	}

	/* 选项 */
	.options {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}
	.option-btn {
		background: #FFFFFF;
		padding: 28rpx 30rpx;
	}
	.option-text {
		font-size: 34rpx;
		font-weight: 800;
		color: #333;
	}
	.option-btn.correct { background: #95E065; }
	.option-btn.wrong { background: #FF5252; }
	.option-btn.wrong .option-text { color: #FFF; }
	.option-btn.dim { opacity: 0.55; }

	/* 详情气泡 */
	.detail-bubble {
		background: #FFF3C4;
		display: flex;
		flex-direction: column;
		gap: 18rpx;
	}
	.word-info-row {
		display: flex;
		align-items: baseline;
		gap: 16rpx;
		flex-wrap: wrap;
	}
	.detail-word {
		font-size: 44rpx;
		font-weight: 900;
		color: #FF6B9D;
		text-shadow: 2rpx 2rpx 0 #FFF;
	}
	.detail-phonetic {
		font-size: 26rpx;
		color: #5D4037;
		font-weight: 700;
		font-style: italic;
	}
	.detail-pos {
		font-size: 22rpx;
		color: #FFF;
		background: #4ECDC4;
		border: 2rpx solid #333;
		border-radius: 10rpx;
		padding: 4rpx 12rpx;
		font-weight: 700;
	}
	.detail-translation {
		font-size: 30rpx;
		font-weight: 800;
		color: #333;
	}
	.audio-row {
		display: flex;
		gap: 16rpx;
	}
	.audio-btn {
		flex: 1;
		background: #4ECDC4;
		padding: 18rpx 0;
	}
	.audio-btn .btn-text {
		color: #FFF;
		font-size: 28rpx;
		font-weight: 800;
		text-shadow: 2rpx 2rpx 0 #333;
	}
	.example-section {
		background: #FFF;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 18rpx 22rpx;
		display: flex;
		flex-direction: column;
		gap: 10rpx;
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
	.extension-card {
		background: #E1F5FE;
		border: 3rpx solid #333;
		border-radius: 14rpx;
		padding: 18rpx 22rpx;
		box-shadow: 4rpx 4rpx 0 #333;
	}
	.extension-text {
		font-size: 26rpx;
		color: #0D47A1;
		font-weight: 600;
		line-height: 1.5;
	}

	/* 上一个 / 下一个 导航 */
	.nav-row {
		display: flex;
		gap: 20rpx;
	}
	.nav-btn {
		flex: 1;
		padding: 26rpx 0;
	}
	.prev-btn {
		background: #FFFFFF;
	}
	.next-btn {
		background: #95E065;
	}
	.nav-btn .btn-text {
		font-size: 32rpx;
		font-weight: 900;
		color: #333;
	}
	/* 置灰不可点状态 */
	.nav-btn.disabled {
		background: #DDD !important;
		opacity: 0.65;
		pointer-events: none;
		box-shadow: 4rpx 4rpx 0 #999;
	}
	.nav-btn.disabled .btn-text {
		color: #888;
	}

	/* 词汇管理弹窗 */
	.vocab-mask {
		position: fixed;
		top: 0; left: 0; right: 0; bottom: 0;
		background: rgba(0, 0, 0, 0.5);
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.3s;
		z-index: 99;
	}
	.vocab-mask.show {
		opacity: 1;
		pointer-events: auto;
	}
	.vocab-sheet {
		position: fixed;
		left: 0; right: 0; bottom: 0;
		background: #FFF9E6;
		border-top: 4rpx solid #333;
		border-left: 4rpx solid #333;
		border-right: 4rpx solid #333;
		border-radius: 32rpx 32rpx 0 0;
		box-shadow: 0 -8rpx 0 #333;
		padding: 40rpx 30rpx 60rpx;
		max-height: 70vh;
		overflow-y: auto;
		transform: translateY(100%);
		transition: transform 0.3s ease-out;
		z-index: 100;
	}
	.vocab-sheet.show {
		transform: translateY(0);
	}
	.sheet-handle {
		width: 80rpx;
		height: 10rpx;
		background: #333;
		border-radius: 6rpx;
		margin: 0 auto 24rpx;
	}
	.close-btn {
		position: absolute;
		top: 24rpx;
		right: 24rpx;
		width: 60rpx;
		height: 60rpx;
		background: #FF5252;
		border: 4rpx solid #333;
		border-radius: 50%;
		box-shadow: 4rpx 4rpx 0 #333;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.close-btn:active {
		transform: translate(4rpx, 4rpx);
		box-shadow: 2rpx 2rpx 0 #333;
	}
	.close-text {
		font-size: 40rpx;
		font-weight: 900;
		color: #FFF;
		line-height: 1;
	}
	.vocab-section {
		margin-bottom: 30rpx;
	}
	.section-title {
		font-size: 32rpx;
		font-weight: 800;
		color: #333;
		margin-bottom: 18rpx;
		display: block;
	}
	.search-row {
		display: flex;
		gap: 16rpx;
	}
	.mbe-input {
		flex: 1;
		background: #FFF;
		border: 4rpx solid #333;
		border-radius: 20rpx;
		box-shadow: 6rpx 6rpx 0 #333;
		padding: 22rpx 24rpx;
		font-size: 28rpx;
		color: #333;
		font-weight: 600;
		margin-bottom: 16rpx;
	}
	.ph {
		color: #999;
		font-weight: 600;
	}
	.search-btn {
		background: #4ECDC4;
		padding: 22rpx 32rpx;
	}
	.search-btn .btn-text {
		color: #FFF;
		font-size: 28rpx;
		font-weight: 800;
	}
	.search-result-card {
		background: #FFF;
		margin-top: 8rpx;
	}
	.result-box {
		display: flex;
		flex-direction: column;
		gap: 10rpx;
	}
	.result-word {
		font-size: 40rpx;
		font-weight: 900;
		color: #FF6B9D;
	}
	.result-phonetic {
		font-size: 26rpx;
		color: #5D4037;
		font-style: italic;
		font-weight: 700;
	}
	.result-trans {
		font-size: 30rpx;
		font-weight: 700;
		color: #333;
	}
	.result-example {
		font-size: 26rpx;
		color: #5D4037;
	}
	.result-example-zh {
		font-size: 26rpx;
		color: #795548;
	}
	.result-source {
		font-size: 22rpx;
		color: #888;
		margin-top: 6rpx;
	}
	.not-found {
		font-size: 30rpx;
		font-weight: 700;
		color: #FF5252;
		text-align: center;
		padding: 20rpx 0;
	}
	.add-btn {
		background: #FFD93D;
		margin-top: 8rpx;
	}
	.add-btn .btn-text {
		color: #333;
		font-size: 30rpx;
		font-weight: 800;
	}

	/* 可选词库列表 — 硬编码按钮 */
	.library-buttons {
		display: flex;
		flex-direction: column;
		gap: 16rpx;
	}
	.lib-btn {
		background: #F5F5F5;
		color: #333;
		border: 4rpx solid #333;
		border-radius: 16rpx;
		padding: 26rpx 0;
		text-align: center;
		font-size: 30rpx;
		font-weight: 800;
		box-shadow: 4rpx 4rpx 0 #333;
	}
	.lib-btn:active {
		transform: translate(2rpx, 2rpx);
		box-shadow: 2rpx 2rpx 0 #333;
	}
	.lib-btn.lib-active {
		background: #FFD93D;
		border: 6rpx solid #333;
	}
</style>
