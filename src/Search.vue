<template>
	<div
		v-if="searchFilter"
		class="st-search"
		:class="{
			'st-search-label-invisible':!labelVisible
		}"
	>
		<x-form
			ref="form"
			inline
			:field-list="searchFilter"
			size="small"
			:label-visible="labelVisible"
			@submit="search"
		>
			<div class="st-search-btns">
				<x-button
					:conf="{
						nativeType: 'submit',
						type: 'primary',
						icon: 'st-icon st-icon-search',
						text: locale.search
					}"
				/>
				<x-button
					v-if="searchResetable"
					:conf="{
						cls: 'st-search-reset-btn',
						text: locale.reset
					}"
					@click.prevent="reset"
				/>
				<x-button
					:conf="{
						text: '取消'
					}"
					@click="closepanel"
				/>
			</div>
		</x-form>
	</div>
</template>

<script>
import XButton from './com/Button.vue';
import XForm from './form/index.vue';

export default {
	components:{ XButton, XForm },
	inject: [
		'searchFilter',
		'searchResetable',
		'labelVisible',
		'listeners',
		'actionMethods',
		'urlSearchParams',
		'params',
		'store',
		'ignoreEmptySearchParam',
		'locale'
	],
	methods: {
		search(evt) {
			let searchParams = Object.assign({}, this.urlSearchParams, evt);
			if(this.ignoreEmptySearchParam) {
				searchParams = this.trimParam(searchParams);
				
			}
			let ret = this.store.emit('search', searchParams);
			if(ret===false)
				return;
			this.store.searchParams = searchParams;
			this.store.$emit('load', {reset: true});
			this.closepanel();
		},
		closepanel() {
			this.$emit('closepanel');
		},
		reset() {
			this.$refs.form.reset();
		},
		trimParam(data){
			let params = {};
			for(let key in data) {
				if(typeof data[key]=='string') {
					if(data[key])
						params[key] = data[key];
				} else if(typeof data[key] != 'undefined') {
					params[key] = data[key];
				}
			}

			return params;
		}
	}
};
</script>

<style lang="scss">
.st-search{
	padding: 10px 0 0 10px;

	&-reset-btn{
		margin-left: 10px;
	}
	&-btns {
		width: 100%;
		position: fixed;
		left: 0;
		bottom: 0;
		border-top: 1px solid #e8eaec;
		height: 60px;
		line-height: 60px;
		display: flex;
		align-items: center;
		justify-content: space-around;
	}
}
</style>
