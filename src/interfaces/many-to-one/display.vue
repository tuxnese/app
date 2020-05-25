<template>
	<div :class="['badge', 'no-wrap']" :style="style">{{ displayValue }}</div>
</template>

<script>
import mixin from '@directus/extension-toolkit/mixins/interface';
import { get, has } from 'lodash';

export default {
	mixins: [mixin],
	data() {
		return {
			loading: false,
			data: null // when the primary key is passed in, we'll fetch the related item so we can
			// display the value like normal
		};
	},
	computed: {
		displayValue() {
			let value = this.value;

			if (this.isPrimaryKey && this.data && this.loading === false) {
				value = this.data;
			}

			if (value) {
				return this.$helpers.micromustache.render(this.options.template, value);
			}

			return '--';
		},
		style() {
			let value = this.value;
			let background_color = 'inherit';
			let text_color = 'inherit';

			if (
				!!this.options.backgroundColorField &&
				has(value, this.options.backgroundColorField) &&
				get(value, this.options.backgroundColorField) != ''
			) {
				background_color = get(value, this.options.backgroundColorField);
			}
			if (
				!!this.options.textColorField &&
				has(value, this.options.textColorField) &&
				get(value, this.options.textColorField) != ''
			) {
				text_color = get(value, this.options.textColorField);
			}
			return {
				backgroundColor: background_color,
				color: text_color
			};
		},
		isPrimaryKey() {
			return typeof this.value !== 'object';
		}
	},
	watch: {
		value() {
			if (this.isPrimaryKey) {
				this.fetchRelationalData();
			}
		}
	},
	methods: {
		async fetchRelationalData() {
			if (this.relation?.collection_one?.collection) {
				this.loading = true;

				const res = await this.$api.getItem(
					this.relation.collection_one.collection,
					this.value
				);

				this.data = res.data;

				this.loading = false;
			}
		}
	}
};
</script>

<style lang="scss" scoped>
.badge {
	border-radius: var(--border-radius);
	padding: 5px 8px 4px;
	font-weight: var(--weight-bold);
	display: block;
	cursor: default;
	max-width: max-content;
	text-overflow: ellipsis;
}
</style>
