<template>
		<!-- @dragenter="emitEvent(events.dragenter, $event)" -->
		<!-- @dragleave="emitEvent(events.dragleave, $event)" -->
		<!-- @dragover.prevent="emitEvent(events.dragover, $event)" -->
		<!-- @drop.prevent="emitEvent(events.drop, $event)" -->
	<component :is="tag" ref="drop">
		<slot></slot>
	</component>
</template>

<script>
import transferDataStore from './transferDataStore';
import { events } from './constants';

export default {
	props: {
		tag: { type: String, default: 'div' },
	},
	data() {
		return {
			mousePos: { x:0, y:0 },
			hovering: false,
		};
	},
	computed: {
		events: () => events,
	},
	methods: {
		onWindowMouseUp() {
			if (this.hovering) {
				this.$emit('drop', transferDataStore.data);
			}
		},
		onWindowMouseMove(event) {
			this.mousePos.x = event.pageX;
			this.mousePos.y = event.pageY;
			const { top, left, right, bottom } = this.$refs.drop.getBoundingClientRect();

			this.hovering = this.mousePos.x >= left
				&& this.mousePos.x < right
				&& this.mousePos.y >= top
				&& this.mousePos.y < bottom;
		},
	},
	mounted() {
		window.addEventListener('mouseup', this.onWindowMouseUp);
		window.addEventListener('mousemove', this.onWindowMouseMove);
	},
	destroyed() {
		window.removeEventListener('mouseup', this.onWindowMouseUp);
		window.removeEventListener('mousemove', this.onWindowMouseMove);
	},
};
</script>
