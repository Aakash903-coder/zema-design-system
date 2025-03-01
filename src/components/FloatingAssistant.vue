<template>
	<div
		:id="id"
		class="floating-assistant"
		:class="{ 'floating-assistant--hidden': !isActive }"
	>
		<rds-pulsar
			:id="pulsarId"
			:target-id="targetId"
			:variant="variant"
		/>
		<div
			:id="containerId"
			v-rds-floatify:[position]="pulsarId"
			class="floating-assistant__container"
		>
			<div
				v-on-click-outside="collapse"
				class="floating-assistant__dropdown"
				:class="{
					'floating-assistant__dropdown--expanded': isExpanded,
					'floating-assistant__dropdown--confirmation': waitingConfirmation,
				}"
				@click="expand"
			>
				<div v-if="waitingConfirmation">
					Hide tip forever?
					<span
						:class="`floating-assistant__link--${variant}`"
						@click="confirmationHandle(true)"
					>
						Yes
					</span> /
					<span
						:class="`floating-assistant__link--${variant}`"
						@click="confirmationHandle(false)"
					>
						No
					</span>
				</div>
				<div v-else>
					<span :class="`floating-assistant__title--${variant}`">
						{{ title }}
					</span>
					<div
						v-if="isExpanded"
						class="floating-assistant__content"
					>
						<!-- @slot Slot used to insert content into the FloatingAssistant card
							when it is expanded -->
						<slot />
						<span class="floating-assistant__footer">
							You can find out more
							<a
								:class="`floating-assistant__link--${variant}`"
								:href="url"
								target="_blank"
							>
								clicking here
							</a>.
						</span>
					</div>
					<span
						v-else
						class="floating-assistant__subtitle"
					>
						Click for more information
					</span>
				</div>
				<div v-if="isExpanded">
					<div @click.stop="close">
						<rds-icon
							class="floating-assistant__close-button"
							name="x-outline"
							height="20"
							width="20"
						/>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import vClickOutside from 'click-outside-vue3';
import RdsPulsar from './Pulsar.vue';
import RdsIcon from './Icon.vue';

export default {
	components: {
		RdsPulsar,
		RdsIcon,
	},

	directives: {
		'on-click-outside': vClickOutside.directive,
	},

	props: {
		/**
		 * Represents the state of the component. When 'false' the component is not shown
		 * and when 'true' the component is displayed.
		 */
		modelValue: {
			type: Boolean,
			default: true,
			required: true,
		},
		/**
		 * The title of the floating card that is displayed.
		 */
		title: {
			type: String,
			default: 'New functionality',
			validator: (value) => value.length <= 22,
		},
		/**
		 * Defines whether the component's display animation will only start after the user scrolls.
		 */
		startOnScroll: {
			type: Boolean,
			default: false,
			required: false,
		},
		/**
		 * The url to redirect to an external page when clicking on the
		 * 'click here' to find out more about what is described in the card
		 */
		url: {
			type: String,
			default: '',
			required: true,
		},
		/**
		* The Badge variant. There are 9 variants: 'turquoise', 'green', 'blue',
		* 'violet', 'pink', 'red', 'orange', 'amber' and 'gray'.
		*/
		variant: {
			type: String,
			default: 'green',
		},
		/**
		* Id of the element that will be a reference for the FloatingAssistant rendering.
		*/
		targetId: {
			type: String,
			default: '',
		},
	},

	data() {
		return {
			isActive: this.modelValue,
			id: null,
			pulsarId: null,
			containerId: null,
			position: 'right',
			isExpanded: false,
			waitingConfirmation: false,
			boxTop: 0,
		};
	},

	watch: {
		isActive(newValue) {
			/**
			* Event emitted when the component is displayed ('true') or hidden ('false').
			* @event expanded
			* @type {Event}
			*/
			this.$emit('update:modelValue', newValue);
		},

		modelValue(newValue) {
			// console.log('v-model', newValue);
			this.isActive = newValue;
		},
	},

	mounted() {
		this.handleEventAnimation();
	},

	methods: {
		handleEventAnimation() {
			this.id = `floating-assistant$-${this._uid}`;
			this.pulsarId = `floating-assistant-pulsar$-${this._uid}`;
			this.containerId = `floating-assistant-container$-${this._uid}`;
			if (this.startOnScroll) {
				window.addEventListener('scroll', this.startAnimation);
			} else {
				this.startAnimation();
			}
		},

		startAnimation() {
			if (this.startOnScroll) {
				if (this.isScrolledIntoView()) {
					setTimeout(() => {
						document.getElementById(this.id).classList.add('animation');
						window.removeEventListener('scroll', this.startAnimation);
					}, 1000);
				}
			} else {
				setTimeout(() => {
					document.getElementById(this.id).classList.add('animation');
				}, 1000);
			}
		},

		isScrolledIntoView() {
			const { top, bottom } = document.getElementById(this.id).getBoundingClientRect();

			return (top >= 0 && bottom <= window.innerHeight);
		},

		expand() {
			if (!this.waitingConfirmation) {
				this.isExpanded = true;
			}
		},

		collapse() {
			this.isExpanded = false;
		},

		close() {
			this.collapse();
			this.waitingConfirmation = true;
		},

		confirmationHandle(shouldDisable) {
			if (!shouldDisable) {
				this.isActive = false;
				return;
			}

			/**
			* Event indicating that the option to disable the tip was selected
			* @event disable-tip
			* @type {Event}
			*/
			this.$emit('disable-tip', true);
			this.isActive = false;
		}
	},
};
</script>


<style lang="scss" scoped>
@import '../assets/sass/tokens.scss';

.floating-assistant {
	&.animation {
		.floating-assistant__dropdown {
			display: flex;
			animation-name: showCard;
			animation-duration: 0.5s;
			animation-fill-mode: forwards;

			&--expanded {
				animation-name: expandCard;
				animation-duration: 0.5s;
				animation-fill-mode: forwards;
			}
		}

		.floating-assistant__title {
			animation: fadeInTitle ease 1s;
			animation-iteration-count: 1;
			animation-fill-mode: forwards;
		}
	}

	&--hidden {
		animation: fadeOut ease .2s;
		animation-iteration-count: 1;
		animation-fill-mode: forwards;
	}

	&__container {
		display: flex;
		align-items: center;
		width: 100%;
	}

	&__dropdown {
		@include caption;
		display: flex;
		color: $n-600;
		background-color: $n-0;
		border-radius: $border-radius-small;
		outline: 1px solid $n-20;
		box-shadow: $shadow-md;
		position: absolute;
		margin: ml(3);
		padding: pYX(2, 5);
		z-index: $z-index-tooltip;
		max-width: 400px;
		transition : 0.3s ease-in-out;

		&--expanded {
			padding: pYX(3, 5);
			width: 30%;
			max-height: none;

			.floating-assistant__title {
				animation: fadeInContent ease 1s;
				animation-iteration-count: 1;
				animation-fill-mode: forwards;
			}
		}

		&--confirmation {
			animation-name: collapseCard;
			animation-duration: 5s;
			animation-fill-mode: forwards;

			span {
				font-weight: $font-weight-bold;
			}
		}
	}

	&__title {
		@include variantResolver using ($color-name, $shade-50, $shade-100, $shade-200, $shade-300, $base-color, $shade-500, $shade-600) {
			width: max-content;
			@include caption;
			color: $shade-500;
			font-weight: $font-weight-bold;
			width: 162px;
			text-overflow: ellipsis;
			overflow: hidden;
			display: -webkit-box;
			-webkit-line-clamp: 2;
			-webkit-box-orient: vertical;
			margin: mb(2);
		}
	}

	&__subtitle {
		display: block;
		width: max-content;
		color: $n-400;

		animation: fadeInTitle ease 1s;
		animation-iteration-count: 1;
		animation-fill-mode: forwards;
	}

	&__content {
		margin: mt(1);
		width: 304px;
		line-height: 132%;
		animation: fadeInContent ease 1s;
		animation-iteration-count: 1;
		animation-fill-mode: forwards;
	}

	&__footer {
		display: block;
		margin: mt(2);
	}

	&__link {
		@include variantResolver using ($color-name, $shade-50, $shade-100, $shade-200, $shade-300, $base-color, $shade-500, $shade-600) {
			color: $shade-500;
			cursor: pointer;
		}
	}

	&__close-button {
		color: $n-600;
		cursor: pointer;

		animation: fadeInContent ease 2s;
		animation-iteration-count: 1;
		animation-fill-mode: forwards;
	}
}

@keyframes fadeInContent {
	0% {
		opacity: 0;
	}
	50% {
		opacity: 0.3;
	}
	100% {
		opacity: 1;
	}
}

@keyframes fadeInTitle {
	0% {
		opacity: 0;
		margin-top: 8px;
	}
	50% {
		opacity: 0.3;
		margin-top: 2px;
	}
	100% {
		opacity: 1;
		margin-top: 0;
	}
}

@keyframes showCard {
	from { width: 0; }
	to { width: 220px; }
}

@keyframes expandCard {
	from {
		width: 220px;
	}
	to {
		width: 356px;
	}
}

@keyframes collapseCard {
	from {
		width: 356px;
	}
	to {
		width: 206px;
	}
}

@keyframes fadeOut {
	0% {
		opacity: 1;
	}
	50% {
		opacity: 0.3;
	}
	100% {
		opacity: 0;
		display: none;
	}
}
</style>