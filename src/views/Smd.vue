<template>
	<div class="smd-page">
		<div class="container-fluid">
			<div class="row">
				<div class="col-12">
					<Card>
						<template #title>
							{{ getNameByKey('smd') }}
						</template>
						<template #body>
							<div class="smd-content">
								<h4>SMD Preview</h4>
								<p>This is a preview page for SMD functionality.</p>
								
								<div class="card mt-4">
									<div class="card-body">
										<h5 class="card-title">SMD Features</h5>
										<ul class="list-group list-group-flush">
											<li class="list-group-item">Feature 1: SMD Data Visualization</li>
											<li class="list-group-item">Feature 2: SMD Analytics</li>
											<li class="list-group-item">Feature 3: SMD Reporting</li>
										</ul>
									</div>
								</div>
							</div>
						</template>
					</Card>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import Card from '@/components/containers/Card.vue';

export default {
	components: {
		Card
	},

	computed: {
		getNameByKey() {
			return this.$store.getters['ui/getNameByKey'];
		}
	}
};
</script>

<style lang="scss" scoped>
.smd-page {
	padding: 20px 0;
}

.smd-content {
	padding: 20px;
}

.card {
	border: 1px solid var(--card-border);
	background-color: var(--card-bg);
}

.list-group-item {
	background-color: transparent;
	border-color: var(--card-border);
	color: var(--primary-text);
}
</style>