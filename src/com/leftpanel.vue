<template>
	<div :class="['st-leftpanel', showPanel ? 'st-leftpanel-show' : '']">
		<x-button 
			:conf="{
				text: '筛选',
				cls: 'st-leftpanel-search-btn'
			}"
			@click="triggerClick" />
		<span>
			<x-search />
		</span>
	</div>
</template>

<script>
import XButton from './Button.vue';
import XSearch from '../Search.vue';
export default {
	components: {XButton, XSearch},
	data() {
		return {
			showPanel: false
		};
	},
	methods: {
		triggerClick() {
			this.showPanel = true;
		}
	}
};
</script>
<style lang="scss">
.st-leftpanel {
	width: 100%;
	height: 100%;
	position: fixed;
	background-color: rgba(0, 0, 0, 0.4);
	z-index: 9;
	transform: translateX(-100%);
	transition: all 0.3s linear;
	&-search-btn {
		position: absolute;
		right: -100%;
	}
	&-show {
		transform: translateX(0);
	}
}
</style>
