<template>
	<Card>
		<template #title>
			{{ title }}
		</template>
		<template #body>
			<div class="smd-widget">
				<div v-if="loading" class="loading-container">
					<Loading />
				</div>
				
				<div v-else>
					<div v-if="socialMetadata.length === 0" class="no-data">
						<p>{{ getNameByKey('noDataFound') || 'No social metadata found' }}</p>
					</div>

					<div v-else class="smd-grid">
						<div 
							v-for="(item, index) in socialMetadata" 
							:key="index"
							class="smd-card"
							@click="openUrl(item.url)"
						>
							<div class="smd-card-image">
								<img 
									v-if="item.imageUrl && !getImageErrorStatus(item.id)" 
									:src="item.imageUrl" 
									:alt="item.name"
									@error="handleImageError($event, item.id)"
									@load="handleImageLoad(item.id)"
								>
								<div v-else class="placeholder-image">
									<img :src="accountIcon" :alt="item.name || 'Account'" class="account-icon">
								</div>
							</div>
							<div class="smd-card-content">
								<h5 class="smd-card-title">{{ item.name || 'Unnamed' }}</h5>
								<p class="smd-card-url">{{ item.url || 'No URL' }}</p>
								<p v-if="item.namespace" class="smd-card-namespace">
									Namespace: {{ item.namespace }}
								</p>
							</div>
						</div>
					</div>
				</div>
			</div>
		</template>
	</Card>
</template>

<script>
import Card from '@/components/containers/Card.vue';
import Loading from '@/components/Loading.vue';
import http from '@/infrastructure/http';
import IconAccounts from '@/styles/img/account.png';

export default {
	components: {
		Card,
		Loading
	},

	props: {
		title: {
			type: String,
			default: 'Social Metadata'
		}
	},

	data() {
		return {
			imageErrors: [], // 画像読み込みエラーを追跡
			accountIcon: IconAccounts // アカウントアイコン
		};
	},

	computed: {
		getNameByKey() {
			return this.$store.getters['ui/getNameByKey'];
		},
		
		socialMetadata() {
			return this.$store.getters['socialMetadata/getSocialMetadata'];
		},
		
		loading() {
			return this.$store.getters['socialMetadata/isLoading'];
		}
	},

	mounted() {
		// NODE_URLが利用可能になったら初期データを取得
		this.$nextTick(async () => {
			// http.nodeUrlが利用可能になるまで待機
			let attempts = 0;
			const maxAttempts = 10;
			
			const waitForNodeUrl = async () => {
				if (http.nodeUrl && this.socialMetadata.length === 0 && !this.loading) {
					await this.$store.dispatch('socialMetadata/fetchSocialMetadata');
					return;
				}
				
				attempts++;
				if (attempts < maxAttempts) {
					setTimeout(waitForNodeUrl, 500);
				} else {
					console.warn('NODE_URL not available after maximum attempts');
				}
			};
			
			await waitForNodeUrl();
		});
	},

	methods: {
		openUrl(url) {
			if (url) {
				window.open(url, '_blank');
			}
		},

		handleImageError(event, itemId) {
			// 画像エラーを記録
			if (!this.imageErrors.includes(itemId)) {
				this.imageErrors.push(itemId);
			}
			console.warn('Image load failed for:', itemId);
		},

		handleImageLoad(itemId) {
			// 画像が正常に読み込まれた場合、エラー状態をクリア
			const index = this.imageErrors.indexOf(itemId);
			if (index > -1) {
				this.imageErrors.splice(index, 1);
			}
		},

		getImageErrorStatus(itemId) {
			return this.imageErrors.includes(itemId);
		}
	}
};
</script>

<style lang="scss" scoped>
.smd-widget {
	padding: 20px;
}

.loading-container {
	display: flex;
	justify-content: center;
	padding: 40px 0;
}

.no-data {
	text-align: center;
	padding: 40px 0;
	color: var(--secondary-text);
}

.smd-grid {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
	gap: 20px;
	margin-top: 20px;
}

.smd-card {
	border: 1px solid var(--card-border);
	border-radius: 8px;
	padding: 16px;
	background-color: var(--card-bg);
	cursor: pointer;
	transition: all 0.2s ease;
	
	&:hover {
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
		border-color: var(--primary);
	}
}

.smd-card-image {
	width: 64px;
	height: 64px;
	border-radius: 50%;
	overflow: hidden;
	margin-bottom: 12px;
	background-color: #140202;
	
	img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}
	
	.placeholder-image {
		width: 100%;
		height: 100%;
		background: linear-gradient(135deg, var(--primary) 0%, var(--primary-hover) 100%);
		color: white;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	.account-icon {

		padding: 8px;
        background-color: rgb(22 23 26 / 0%);
		border-radius: 50%;
		filter: brightness(0) invert(1); /* 白色にする */
	}
}

.smd-card-title {
	font-size: 16px;
	font-weight: 600;
	margin-bottom: 8px;
	color: var(--primary-text);
}

.smd-card-url {
	font-size: 14px;
	color: var(--link-text);
	margin-bottom: 4px;
	word-break: break-all;
}

.smd-card-namespace {
	font-size: 12px;
	color: var(--secondary-text);
	margin: 0;
}
</style>