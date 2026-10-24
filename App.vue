<script>
	export default {
		onLaunch: function() {
			// 幂等初始化：缺啥补啥，已存在的值保留
			// 注意：不检查 null —— 破壳后 currentEgg 设为 null 表示「已孵化，无蛋」，
			// 不应在下次启动时被重新初始化为「未知的蛋」。
			const ensure = (key, value) => {
				const cur = uni.getStorageSync(key);
				if (cur === '' || cur === undefined) {
					uni.setStorageSync(key, value);
				}
			};
			// 路演演示专属版：每次启动强制覆盖演示数据
			// 1. 资源数据
			uni.setStorageSync('points', 200);
			uni.setStorageSync('coins', 50);
			uni.setStorageSync('food', 150);
			uni.setStorageSync('potions', 5);
			uni.setStorageSync('magicPotions', 5);
			// 2. 动物列表（含正常/生病/休眠三种状态 + nickname + source 字段）
			uni.setStorageSync('animals', [
				// 动物1：高级-正常
				{ name: '小狐狸', nickname: '小狐狸', emoji: '🦊', stage: '成年', growth: 0, status: 'normal', source: '高级', lastFeedTime: Date.now() },
				// 动物2：初级-生病（lastFeedTime 2 天前，符合生病逻辑）
				{ name: '小鸡', nickname: '小鸡', emoji: '🐤', stage: '幼年', growth: 0, status: 'sick', source: '初级', lastFeedTime: Date.now() - 2 * 24 * 3600 * 1000 },
				// 动物3：中级-休眠（lastFeedTime 4 天前，符合休眠逻辑）
				{ name: '小猫', nickname: '小猫', emoji: '🐱', stage: '青年', growth: 0, status: 'sleep', source: '中级', lastFeedTime: Date.now() - 4 * 24 * 3600 * 1000 }
			]);
			// 3. 孵化队列：放入一枚特级蛋
			uni.setStorageSync('eggsList', [
				{ id: 9999, name: '未知的蛋', level: '特级', animalName: '大老虎', emoji: '🐯', progress: 10, source: '特级' }
			]);
			// 4. 激活的蛋 ID：指向上面那枚特级蛋
			uni.setStorageSync('activeEggId', 9999);
			// 其余 key 保留 ensure（不每次覆盖）
			ensure('streak', 1);
			ensure('currentLibrary', 'CET-4');
			ensure('myWords', []);

			// 版本迁移
			const ver = uni.getStorageSync('appVersion');
			if (ver !== '4') {
				// v3 → v4：补全动物新字段
				const animals = uni.getStorageSync('animals');
				if (Array.isArray(animals)) {
					animals.forEach(a => {
						if (typeof a.lastFeedTime !== 'number') a.lastFeedTime = Date.now();
						if (!a.status) a.status = 'normal';
					});
					uni.setStorageSync('animals', animals);
				}
				// 清理 v1 残留 key
				uni.removeStorageSync('lastStudyDate');
				uni.removeStorageSync('growth');
				uni.removeStorageSync('evolved');
				uni.removeStorageSync('totalLearned');
				uni.removeStorageSync('hatched');
				uni.removeStorageSync('hatchedPet');
				uni.setStorageSync('appVersion', '4');
			}
		},
		onShow: function() {},
		onHide: function() {}
	}
</script>

<style>
	/* 全局 MBE 基础样式 */
	page {
		background: #FFF9E6;
		font-family: -apple-system, "PingFang SC", "Microsoft Yahei", sans-serif;
		color: #333;
	}

	/* MBE 卡片：粗边框 + 硬阴影 + 大圆角 */
	.mbe-card {
		border: 4rpx solid #333;
		border-radius: 24rpx;
		box-shadow: 8rpx 8rpx 0px #333;
		padding: 30rpx;
		box-sizing: border-box;
	}

	/* MBE 按钮：按压时向右下平移 + 阴影变小（MBE 像素立体感） */
	.mbe-btn {
		border: 4rpx solid #333;
		border-radius: 24rpx;
		box-shadow: 8rpx 8rpx 0px #333;
		padding: 24rpx 40rpx;
		font-weight: 700;
		display: flex;
		justify-content: center;
		align-items: center;
		transition: transform 0.1s, box-shadow 0.1s;
	}
	.mbe-btn:active {
		transform: translate(4rpx, 4rpx);
		box-shadow: 4rpx 4rpx 0px #333;
	}

	.mbe-page {
		min-height: 100vh;
		width: 100%;
		max-width: 100vw;
		overflow-x: hidden;
		/* 顶部安全距离：避免内容紧贴屏幕顶端 */
		padding: 40rpx 30rpx 60rpx;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		gap: 24rpx;
	}
</style>
