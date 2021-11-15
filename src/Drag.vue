<template>
		<!-- :draggable="draggable" -->
		<!-- @drag="emitEvent(events.drag, $event)" -->
		<!-- @dragstart="emitEvent(events.dragstart, $event)" -->
		<!-- @dragenter="emitEvent(events.dragenter, $event)" -->
		<!-- @dragleave="emitEvent(events.dragleave, $event)" -->
		<!-- @dragend="emitEvent(events.dragend, $event)" -->
	<component ref="target" :is="tag"
		@mousedown="emitEvent(events.mousedown, $event)"
		@mouseenter="emitEvent(events.dragmouseenter, $event)"
		@mouseleave="emitEvent(events.dragmouseleave, $event)"
	>
		<slot></slot>
		<div v-if="dragging" class="drag-clone" :style="cloneStyles">
			<slot name="clone"></slot>
		</div>
	</component>
</template>

<script>
import transferDataStore from './transferDataStore';
import { events } from './constants';

export default {
	props: {
		draggable: { type: Boolean, default: true },
		tag: { type: String, default: 'div' },
		transferData: {},
	},
	data() {
		return {
			dragging: false,
			mousedown: false,
			hover: false,
			clonePos: { y:0, x:0 },
		};
	},
	computed: {
		events: () => events,
		cloneStyles() {
			return {
				top: `${this.clonePos.y}px`,
				left: `${this.clonePos.x}px`,
			};
		},
	},
	watch: {
		dragging(newVal, oldVal) {
			if (newVal !== oldVal) {
				if ((! oldVal) && this.dragging) {
					this.onDragStart();
				}
				if (oldVal && (! this.dragging)) {
					this.$emit(events.dragend);
				}
			}
		},
	},
	methods: {
		emitEvent(name) {
			switch (name) {
				case events.mousedown:
					this.onMousedown();
					break;
				case events.dragmouseenter:
					this.onMouseEnter();
					break;
				case events.dragmouseleave:
					this.onMouseLeave();
					break;
				default:
					break;
			}
		},
		onDragStart() {
			transferDataStore.data = this.transferData;
			this.$emit(events.dragstart);
		},
		onDrag(event) {
			const { pageX, pageY } = event;
			this.clonePos.x = pageX;
			this.clonePos.y = pageY;
		},
		onMousedown() {
			this.mousedown = true;
		},
		onMouseEnter() {
			this.hover = true;
		},
		onMouseLeave() {
			this.hover = false;
		},
		onWindowMouseMove(event) {
			this.dragging = ! this.dragging
				? this.mousedown && this.hover
				: this.mousedown;

			if (this.dragging) {
				this.onDrag(event);
			}
		},
		onWindowMouseUp() {
			this.mousedown = false;
			this.hover = false;
			this.dragging = false;
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
