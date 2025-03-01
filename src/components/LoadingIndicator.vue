<template>
	<div :class="`loading-indicator--${variant}`" />
</template>

<script setup>
import { computed, ref, watch } from 'vue';
import isDeviceType from '../utils/methods/isDeviceType.js';

// Props
const props = defineProps({
	/**
	 * Controls the display of the LoadingIndicator.
	 */
	modelValue: {
		type: Boolean,
		default: false,
		required: true,
	},
	/**
	 * The color variant. There are 10 variants implemented: 'green', 'teal',
	 * 'blue', 'indigo', 'violet', 'pink', 'red', 'orange', 'amber' and 'turquoise'.
	 */
	variant: {
		type: String,
		default: 'green',
	},
});

// Reactive variables
const currentPercentage = ref(0);

// Computed variables
const barPercentage = computed(() => {
	return `${currentPercentage.value}%`;
});

const disabledTransition = computed(() => {
	return currentPercentage.value === 0 ? 'none' : 'width 0.5s';
});

const barHeight = computed(() => {
	if (isDeviceType('smartphone') || isDeviceType('tablet')) {
		return '4px';
	}

	return '3px';
})

// Watchers
watch(() => props.modelValue, (newValue) => {
	if (newValue) {
		start();
		increment();
		return;
	}

	finish();
});

// Methods
function start() {
	currentPercentage.value = 1;
}

function increment() {
	if (currentPercentage.value > 0 && currentPercentage.value < 90) {
		currentPercentage.value += Math.floor(Math.random() * (25 - 10 + 1) + 10);

		if (currentPercentage.value >= 90) {
			currentPercentage.value = 90;
			return;
		}

		setTimeout(increment, Math.floor(Math.random() * (1300 - 500 + 1) + 500));
	}
}

function finish() {
	currentPercentage.value = 100;
	setTimeout(() => {
		currentPercentage.value = 0;
	}, 600);
}

</script>

<style lang="scss" scoped>
@import '../assets/sass/tokens.scss';

.loading-indicator {
	position: fixed;
	top: 0;
	left: 0;
	width: v-bind(barPercentage);
	height: v-bind(barHeight);
	display: flex;
	justify-content: center;
	align-items: center;
	background-color: $n-10;
	background-size: 200% 100%;
	animation: loading-gradient 2s ease infinite;

	@keyframes loading-gradient {
		0% {
			background-position: 0% 0%;
		}
		100% {
			background-position: -100% 0%;
		}
	}
	z-index: $z-index-toast;
	transition: v-bind(disabledTransition);

	@include variantResolver using ($color-name, $shade-50, $shade-100, $shade-200, $shade-300, $base-color, $shade-500, $shade-600) {
		@extend .loading-indicator;
		background-image: linear-gradient(to right, rgba($base-color, 0.6), rgba($base-color, 1));
	}
}
</style>