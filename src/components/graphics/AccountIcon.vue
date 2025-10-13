<template>
	<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" :x="_x" :y="_y"
		:width="_width" :height="_height" :viewBox="viewBox" xml:space="preserve" class="account"
		@click="onAccountClick(address)">
		<title> {{ title }} </title>
		<rect x="25.266" y="107.646" fill-rule="evenodd" clip-rule="evenodd" fill="none" width="207.333"
			height="23.667" />
		<text v-if="!hideCaption" x="130" y="122.8457" class="account-text" text-anchor="middle">{{ truncatedAddress
		}}</text>

		<svg x="96px" y="30px">
			<foreignObject width="64" height="64" v-if="imageUrl && !isHidden">
				<div xmlns="http://www.w3.org/1999/xhtml" class="icon-container">
					<a :href="url" target="_blank">
						<img 
							:src="imageUrl" 
							class="circle-image" 
							@error="handleImageError"
							@load="handleImageLoad"
							loading="lazy"
							:alt="`Account icon for ${address}`"
							:class="{ 'loading': isLoading }"
						/>
					</a>
					<!-- Loading indicator -->
					<div v-if="isLoading" class="loading-spinner"></div>
					<button 
						class="hide-icon-btn" 
						@click.stop="hideIcon"
						title="Hide inappropriate icon"
						:aria-label="`Hide icon for ${address}`"
					>
						✕
					</button>
				</div>
			</foreignObject>
			<g v-else>
				<path
					d="M2.5 32C2.5 15.4315 15.9315 2 32.5 2V2C49.0685 2 62.5 15.4315 62.5 32V32C62.5 48.5685 49.0685 62 32.5 62V62C15.9315 62 2.5 48.5685 2.5 32V32Z"
					:fill="iconColor" />
				<path fill-rule="evenodd" clip-rule="evenodd"
					d="M27.6558 24.1875C27.6558 21.5124 29.8244 19.3438 32.4995 19.3438C35.1746 19.3438 37.3433 21.5124 37.3433 24.1875C37.3433 26.8626 35.1746 29.0312 32.4995 29.0312C29.8244 29.0312 27.6558 26.8626 27.6558 24.1875ZM32.4995 16.5312C28.2711 16.5312 24.8433 19.9591 24.8433 24.1875C24.8433 28.4159 28.2711 31.8437 32.4995 31.8437C36.7279 31.8437 40.1558 28.4159 40.1558 24.1875C40.1558 19.9591 36.7279 16.5312 32.4995 16.5312ZM22.9683 42.9375C22.9683 40.2624 25.1369 38.0938 27.812 38.0938H37.187C39.8621 38.0938 42.0308 40.2624 42.0308 42.9375C42.0308 43.8867 41.2612 44.6562 40.312 44.6562H24.687C23.7378 44.6562 22.9683 43.8867 22.9683 42.9375ZM27.812 35.2812C23.5836 35.2812 20.1558 38.7091 20.1558 42.9375C20.1558 45.44 22.1845 47.4687 24.687 47.4687H40.312C42.8146 47.4687 44.8433 45.44 44.8433 42.9375C44.8433 38.7091 41.4154 35.2812 37.187 35.2812H27.812Z"
					fill="black" />
				<path
					d="M32.5 60.125C16.967 60.125 4.375 47.533 4.375 32H0.625C0.625 49.6041 14.8959 63.875 32.5 63.875V60.125ZM60.625 32C60.625 47.533 48.033 60.125 32.5 60.125V63.875C50.1041 63.875 64.375 49.6041 64.375 32H60.625ZM32.5 3.875C48.033 3.875 60.625 16.467 60.625 32H64.375C64.375 14.3959 50.1041 0.125 32.5 0.125V3.875ZM32.5 0.125C14.8959 0.125 0.625 14.3959 0.625 32H4.375C4.375 16.467 16.967 3.875 32.5 3.875V0.125Z"
					:fill="iconColor" />
				<!-- Show unhide button for hidden icons -->
				<foreignObject v-if="isHidden && imageUrl" x="22" y="22" width="20" height="20">
					<div xmlns="http://www.w3.org/1999/xhtml">
						<button 
							class="unhide-icon-btn"
							@click.stop="unhideIcon"
							title="Show icon"
						>
							👁️
						</button>
					</div>
				</foreignObject>
			</g>
		</svg>
	</svg>

</template>

<script>
import GraphicComponent from './GraphicComponent.vue';
import Http from '@/infrastructure/http'
import { Address, RepositoryFactoryHttp, MetadataType, UInt64 } from 'symbol-sdk'

// グローバルキャッシュマネージャー
class ImageCacheManager {
	constructor() {
		this.memoryCache = new Map();
		this.localStorageKey = 'accountIconCache';
		this.maxCacheSize = 100; // メモリキャッシュの最大サイズ
		this.cacheExpiry = 24 * 60 * 60 * 1000; // 24時間のキャッシュ有効期限
	}

	// キャッシュキーを生成
	generateCacheKey(address, imageUrl) {
		return `${address}_${this.hashString(imageUrl)}`;
	}

	// 簡単なハッシュ関数
	hashString(str) {
		let hash = 0;
		for (let i = 0; i < str.length; i++) {
			const char = str.charCodeAt(i);
			hash = ((hash << 5) - hash) + char;
			hash = hash & hash; // 32bit整数に変換
		}
		return Math.abs(hash).toString(36);
	}

	// メモリキャッシュから取得
	getFromMemory(cacheKey) {
		const cached = this.memoryCache.get(cacheKey);
		if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
			return cached.data;
		}
		this.memoryCache.delete(cacheKey);
		return null;
	}

	// メモリキャッシュに保存
	setToMemory(cacheKey, data) {
		// キャッシュサイズ制限
		if (this.memoryCache.size >= this.maxCacheSize) {
			const firstKey = this.memoryCache.keys().next().value;
			this.memoryCache.delete(firstKey);
		}

		this.memoryCache.set(cacheKey, {
			data,
			timestamp: Date.now()
		});
	}

	// LocalStorageから取得
	getFromLocalStorage(cacheKey) {
		try {
			const allCache = JSON.parse(localStorage.getItem(this.localStorageKey) || '{}');
			const cached = allCache[cacheKey];
			
			if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
				return cached.data;
			}
			
			// 期限切れのキャッシュを削除
			if (cached) {
				delete allCache[cacheKey];
				localStorage.setItem(this.localStorageKey, JSON.stringify(allCache));
			}
			
			return null;
		} catch (error) {
			console.warn('Failed to read from localStorage cache:', error);
			return null;
		}
	}

	// LocalStorageに保存
	setToLocalStorage(cacheKey, data) {
		try {
			const allCache = JSON.parse(localStorage.getItem(this.localStorageKey) || '{}');
			
			// キャッシュサイズ制限（LocalStorage用）
			const cacheKeys = Object.keys(allCache);
			if (cacheKeys.length >= 50) {
				// 古いキャッシュを削除
				const oldestKey = cacheKeys.reduce((oldest, key) => {
					return allCache[key].timestamp < allCache[oldest].timestamp ? key : oldest;
				});
				delete allCache[oldestKey];
			}

			allCache[cacheKey] = {
				data,
				timestamp: Date.now()
			};

			localStorage.setItem(this.localStorageKey, JSON.stringify(allCache));
		} catch (error) {
			console.warn('Failed to save to localStorage cache:', error);
		}
	}

	// キャッシュから取得（メモリ -> localStorage の順）
	get(cacheKey) {
		// まずメモリキャッシュから
		let cached = this.getFromMemory(cacheKey);
		if (cached) {
			return cached;
		}

		// LocalStorageから取得してメモリキャッシュに保存
		cached = this.getFromLocalStorage(cacheKey);
		if (cached) {
			this.setToMemory(cacheKey, cached);
			return cached;
		}

		return null;
	}

	// キャッシュに保存（メモリ と localStorage 両方）
	set(cacheKey, data) {
		this.setToMemory(cacheKey, data);
		this.setToLocalStorage(cacheKey, data);
	}

	// 期限切れキャッシュをクリーンアップ
	cleanup() {
		// メモリキャッシュのクリーンアップ
		for (const [key, value] of this.memoryCache.entries()) {
			if (Date.now() - value.timestamp >= this.cacheExpiry) {
				this.memoryCache.delete(key);
			}
		}

		// LocalStorageのクリーンアップ
		try {
			const allCache = JSON.parse(localStorage.getItem(this.localStorageKey) || '{}');
			let hasChanged = false;

			for (const key of Object.keys(allCache)) {
				if (Date.now() - allCache[key].timestamp >= this.cacheExpiry) {
					delete allCache[key];
					hasChanged = true;
				}
			}

			if (hasChanged) {
				localStorage.setItem(this.localStorageKey, JSON.stringify(allCache));
			}
		} catch (error) {
			console.warn('Failed to cleanup localStorage cache:', error);
		}
	}

	// キャッシュをクリア
	clear() {
		this.memoryCache.clear();
		try {
			localStorage.removeItem(this.localStorageKey);
		} catch (error) {
			console.warn('Failed to clear localStorage cache:', error);
		}
	}
}

// グローバルキャッシュインスタンス
const imageCache = new ImageCacheManager();

// 定期的なクリーンアップ（10分ごと）
setInterval(() => {
	imageCache.cleanup();
}, 10 * 60 * 1000);

export default {
	extends: GraphicComponent,

	props: {
		width: {
			type: Number,
			default: 261.333
		},

		height: {
			type: Number,
			default: 131.313
		},

		address: {
			type: String,
			required: true
		},

		hideCaption: {
			type: Boolean,
			default: false
		}
	},

	data() {
		return {
			id: this.getId('account-icon'),
			imageUrl: null,
			url: null,
			isHidden: false,
			hiddenIconsKey: 'hiddenAccountIcons',
			isLoading: false,
			hasError: false,
			cacheKey: null
		};
	},
	async mounted() {
		await this.loadAccountIcon();
		this.checkIfHidden();
	},

	watch: {
		address() {
			this.loadAccountIcon();
			this.checkIfHidden();
		}
	},

	methods: {
		async loadAccountIcon() {
			if (this.isLoading) return; // 重複ロードを防ぐ
			
			this.isLoading = true;
			this.hasError = false;
			
			try {
				// まずキャッシュから確認
				const tempCacheKey = `metadata_${this.address}`;
				const cachedMetadata = imageCache.get(tempCacheKey);
				
				if (cachedMetadata) {
					console.log('Using cached metadata for:', this.address);
					this.setImageFromMetadata(cachedMetadata);
					this.isLoading = false;
					return;
				}

				// キャッシュにない場合、APIから取得
				const nodeUrl = Http.nodeUrl;
				const repoFactory = new RepositoryFactoryHttp(nodeUrl);
				const metadataRepo = repoFactory.createMetadataRepository();
				const addressObj = Address.createFromRawAddress(this.address);

				const key = "D6FBBD8C20F5AC1C";//social_meta_data

				const searchCriteria = {
					targetAddress: addressObj,
					metadataType: MetadataType.Account,
					scopedMetadataKey: key
				};

				const metadata = await metadataRepo.search(searchCriteria).toPromise();

				if (metadata && metadata.data && metadata.data.length > 0) {
					const imageMeta = JSON.parse(metadata.data[0].metadataEntry.value);
					
					// メタデータをキャッシュに保存
					imageCache.set(tempCacheKey, imageMeta);
					
					this.setImageFromMetadata(imageMeta);
				} else {
					// メタデータなしもキャッシュ（空のオブジェクト）
					imageCache.set(tempCacheKey, {});
					this.imageUrl = null;
					this.url = null;
				}
			} catch (error) {
				console.warn('Failed to load account metadata:', error);
				this.imageUrl = null;
				this.url = null;
				this.hasError = true;
			} finally {
				this.isLoading = false;
			}
		},

		setImageFromMetadata(imageMeta) {
			if (imageMeta.imageUrl && this.validateImageUrl(imageMeta.imageUrl)) {
				// 画像キャッシュのキーを生成
				this.cacheKey = imageCache.generateCacheKey(this.address, imageMeta.imageUrl);
				
				// 画像キャッシュから確認
				const cachedImage = imageCache.get(this.cacheKey);
				if (cachedImage && cachedImage.imageUrl) {
					console.log('Using cached image for:', this.address);
					this.imageUrl = cachedImage.imageUrl;
					this.url = cachedImage.url;
					return;
				}

				// キャッシュにない場合、画像をプリロード
				this.preloadImage(imageMeta.imageUrl).then(() => {
					this.imageUrl = imageMeta.imageUrl;
					this.url = imageMeta.url;
					
					// 成功した画像をキャッシュに保存
					imageCache.set(this.cacheKey, {
						imageUrl: imageMeta.imageUrl,
						url: imageMeta.url
					});
				}).catch(() => {
					console.warn('Failed to preload image:', imageMeta.imageUrl);
					this.imageUrl = null;
					this.url = null;
				});
			} else {
				this.imageUrl = null;
				this.url = null;
			}
		},

		// 画像をプリロードして読み込み可能性を確認
		preloadImage(imageUrl) {
			return new Promise((resolve, reject) => {
				const img = new Image();
				img.onload = () => resolve();
				img.onerror = () => reject();
				img.src = imageUrl;
			});
		},

		/**
		 * 画像URLのバリデーション
		 */
		validateImageUrl(url) {
			if (!url || typeof url !== 'string') {
				return false;
			}

			// URLの長さ制限
			if (url.length > 2048) {
				console.warn('URL too long:', url.length);
				return false;
			}

			// http/https プロトコルのチェック
			if (!url.startsWith('http://') && !url.startsWith('https://')) {
				console.warn('Invalid protocol in URL:', url);
				return false;
			}

			// 基本的なURL形式のチェック
			try {
				new URL(url);
				return true;
			} catch (error) {
				console.warn('Invalid URL format:', url, error);
				return false;
			}
		},

		/**
		 * アイコンが非表示設定されているかチェック
		 */
		checkIfHidden() {
			if (!this.address) return;
			
			const hiddenIcons = this.getHiddenIcons();
			this.isHidden = hiddenIcons.includes(this.address);
		},

		/**
		 * LocalStorageから非表示リストを取得
		 */
		getHiddenIcons() {
			try {
				const stored = localStorage.getItem(this.hiddenIconsKey);
				return stored ? JSON.parse(stored) : [];
			} catch (error) {
				console.error('Error reading hidden icons from localStorage:', error);
				return [];
			}
		},

		/**
		 * LocalStorageに非表示リストを保存
		 */
		setHiddenIcons(hiddenList) {
			try {
				localStorage.setItem(this.hiddenIconsKey, JSON.stringify(hiddenList));
			} catch (error) {
				console.error('Error saving hidden icons to localStorage:', error);
			}
		},

		/**
		 * アイコンを非表示にする
		 */
		hideIcon() {
			if (!this.address) return;
			
			const hiddenIcons = this.getHiddenIcons();
			if (!hiddenIcons.includes(this.address)) {
				hiddenIcons.push(this.address);
				this.setHiddenIcons(hiddenIcons);
				this.isHidden = true;
				
				console.log('Icon hidden for address:', this.address);
				this.$emit('icon-hidden', this.address);
			}
		},

		/**
		 * アイコンの非表示を解除する
		 */
		unhideIcon() {
			if (!this.address) return;
			
			const hiddenIcons = this.getHiddenIcons();
			const index = hiddenIcons.indexOf(this.address);
			if (index > -1) {
				hiddenIcons.splice(index, 1);
				this.setHiddenIcons(hiddenIcons);
				this.isHidden = false;
				
				console.log('Icon shown for address:', this.address);
				this.$emit('icon-shown', this.address);
			}
		},

		/**
		 * 画像読み込み成功処理
		 */
		handleImageLoad() {
			this.isLoading = false;
			this.hasError = false;
		},

		/**
		 * 画像読み込みエラー処理
		 */
		handleImageError() {
			console.warn('Failed to load account icon:', this.imageUrl);
			
			// エラーになった画像のキャッシュを削除
			if (this.cacheKey) {
				imageCache.memoryCache.delete(this.cacheKey);
			}
			
			this.imageUrl = null;
			this.isLoading = false;
			this.hasError = true;
		}
	},
	computed: {
		title() {
			return this.getTranslation('address') + ': ' + this.address;
		},

		iconColor() {
			return this.getIconColor(this.address);
		},

		truncatedAddress() {
			return this.truncString(this.address);
		},

		viewBox() {
			return this.hideCaption
				? '95 30 70 65'
				: '0 0 261.333 131.313';
		}
	},

	// コンポーネント破棄時のクリーンアップ
	beforeDestroy() {
		// 必要に応じて特定のキャッシュをクリア
		// imageCache.memoryCache.delete(this.cacheKey);
	}
};
</script>

<style lang="scss" scoped>
.account {
	cursor: pointer;
}

.account-text {
	font-size: 18px;
	font-weight: bold;
	fill: var(--clickable-text);
}

.circle-image {
	width: 64px;
	height: 64px;
	border-radius: 50%;
	object-fit: cover;
	transition: opacity 0.3s ease;
}

.circle-image.loading {
	opacity: 0.7;
}

.icon-container {
	position: relative;
	display: inline-block;
}

/* ローディングスピナー */
.loading-spinner {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 20px;
	height: 20px;
	border: 2px solid #f3f3f3;
	border-top: 2px solid #3498db;
	border-radius: 50%;
	animation: spin 1s linear infinite;
	z-index: 5;
}

@keyframes spin {
	0% { transform: translate(-50%, -50%) rotate(0deg); }
	100% { transform: translate(-50%, -50%) rotate(360deg); }
}

/* モデレーションボタン */
.hide-icon-btn {
	position: absolute;
	top: -3px;
	right: -3px;
	width: 18px;
	height: 18px;
	border-radius: 50%;
	background: rgba(255, 0, 0, 0.9);
	color: white;
	border: none;
	font-size: 12px;
	line-height: 1;
	cursor: pointer;
	opacity: 0;
	transition: opacity 0.2s ease;
	z-index: 10;
	display: flex;
	align-items: center;
	justify-content: center;
}

.icon-container:hover .hide-icon-btn {
	opacity: 1;
}

/* 非表示解除ボタン */
.unhide-icon-btn {
	width: 20px;
	height: 20px;
	background: rgba(0, 123, 255, 0.8);
	color: white;
	border: none;
	border-radius: 50%;
	cursor: pointer;
	font-size: 10px;
	display: flex;
	align-items: center;
	justify-content: center;
	transition: background 0.2s ease;
}

.unhide-icon-btn:hover {
	background: rgba(0, 123, 255, 1);
}

/* レスポンシブ対応 */
@media (max-width: 768px) {
	.hide-icon-btn {
		width: 20px;
		height: 20px;
		font-size: 14px;
		opacity: 1; /* モバイルでは常に表示 */
	}
	
	.icon-container {
		/* モバイルでのタッチ操作を考慮 */
		-webkit-tap-highlight-color: transparent;
	}
	
	.circle-image {
		width: 48px;
		height: 48px;
	}
}

@media (max-width: 480px) {
	.circle-image {
		width: 40px;
		height: 40px;
	}
	
	.hide-icon-btn {
		width: 16px;
		height: 16px;
		font-size: 12px;
	}
}

/* パフォーマンス最適化 */
.circle-image {
	/* GPU加速を有効にしてスムーズなトランジション */
	will-change: opacity;
	transform: translateZ(0);
}

.icon-container {
	/* ハードウェア加速を有効化 */
	transform: translateZ(0);
}
</style>
