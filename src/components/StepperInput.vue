<template>
	<div>
		<span>
			<span
				v-if="hasSlots"
			>
				<slot name="label" />
			</span>

			<label
				v-else
				class="stepper-input__label"
			>
				<div
					class="label__content"
					for="stepper-input"
				>
					<span>
						{{ label }}
					</span>

					<span
						v-if="required"
						class="label__required-indicator"
					>
						*
					</span>
				</div>
			</label>
		</span>

		<mobile-stepper-input
			v-if="mobile"
			v-model="internalValue"
			:disabled="disabled"
			:state="state"
			@minus="changeValue(-1)"
			@plus="changeValue(1)"
		/>
		<div
			v-else
			:class="stepperInputDynamicClass"
		>
			<input
				id="stepper-input"
				v-model="internalValue"
				:disabled="disabled"
				class="stepper-input__field"
				:class="{
					'stepper-input__field--thin': width === 'thin',
					'stepper-input__field--default': width === 'default',
					'stepper-input__field--wide': width =='wide',
				}"
				type="number"
				@focus="isBeingFocused = true"
				@blur="isBeingFocused = false"
			>

			<div class="stepper-input__icon-container">
				<button
					:disabled="disabled"
					class="stepper-input__icon--plus"
					tabindex="-1"
					@focus="isBeingFocused = true"
					@blur="isBeingFocused = false"
					@mousedown="startLongPress(1)"
					@mouseup="stopLongPress"
					@mouseleave="stopLongPress"
					@touchstart.prevent="startLongPress(1)"
					@touchend="stopLongPress"
					@click="changeValue(1)"
				>
					<rds-icon
						height="16"
						width="16"
						name="plus-outline"
					/>
				</button>

				<button
					:disabled="disabled"
					class="stepper-input__icon--minus"
					tabindex="-1"
					@focus="isBeingFocused = true"
					@blur="isBeingFocused = false"
					@mousedown="startLongPress(-1)"
					@mouseup="stopLongPress"
					@mouseleave="stopLongPress"
					@touchstart.prevent="startLongPress(-1)"
					@touchend="stopLongPress"
					@click="changeValue(-1)"
				>
					<rds-icon
						height="16"
						width="16"
						name="minus-outline"
					/>
				</button>
			</div>
		</div>
	</div>

	<div
		v-if="errorState && !disabled"
		class="stepper-input__error-message"
	>
		{{ errorMessage }}
	</div>
</template>

<script>
import stateValidator from '../utils/validators/state';
import MobileStepperInput from './MobileStepperInput.vue';
import RdsIcon from './Icon.vue';

export default {

	components: {
		RdsIcon,
		MobileStepperInput,
	},

	directives: {
	},

	props: {
		/**
		* Prop used as v-model. Controls the visibility of the popover.
		*/
		modelValue: {
			type: [Number, String],
			default: 0,
		},
		/**
		 * Specifies the largest value that StepperInput should accept.
		 */
		max: {
			type: [Number, String],
			default: Number.MAX_SAFE_INTEGER,
		},
		/**
		 * Specifies the smallest value that StepperInput should accept.
		 */
		min: {
			type: [Number, String],
			default: Number.MIN_SAFE_INTEGER,
		},
		/**
		 * Specifies the input label.
		 */
		label: {
			type: String,
			default: 'Label',
		},
		/**
		 * Displays mandatory asterisk (note: does not validate)
		 */
		required: {
			type: Boolean,
			default: false,
		},
		/**
		 * Defines the width of the Select. The options are 'thin', 'default' and 'wide'.
		 */
		width: {
			type: String,
			default: 'default',
			required: false,
		},
		/**
		 * Disables input.
		 */
		disabled: {
			type: Boolean,
			default: false,
		},
		/**
		 * Specifies the state of StepperInput. The options are 'default', 'valid' and 'invalid'.
		 */
		state: {
			type: String,
			default: 'default',
			validator: stateValidator,
		},
		/**
		 * Specifies the error message, which will be displayed if the status is invalid
		 */
		errorMessage: {
			type: String,
			default: 'Invalid value',
		},
		/**
		 * Indicates whether the component should behave with a focus on mobile applications
		 */
		mobile: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			internalValue: this.modelValue,
			isBeingFocused: false,
			longPressTimer: null,
			longPressInterval: null,
		};
	},

	computed: {
		hasSlots() {
			return !!Object.keys(this.$slots).length;
		},

		errorState() {
			return this.state === 'invalid';
		},

		stepperInputDynamicClass() {
			let stepperInputClass = '';

			if (!this.isBeingFocused) {
				stepperInputClass = 'stepper-input';

				if (!this.disabled) {
					if (this.state === 'valid') {
						stepperInputClass += ' stepper-input--valid';
					} else if (this.state === 'invalid') {
						stepperInputClass += ' stepper-input--invalid';
					}
				} else {
					stepperInputClass += ' stepper-input--disabled';
				}
			} else if (!this.disabled) {
				if (this.state === 'default') {
					stepperInputClass += ' stepper-input--focused';
				} else if (this.state === 'valid') {
					stepperInputClass += ' stepper-input--focused-valid';
				} else if (this.state === 'invalid') {
					stepperInputClass += ' stepper-input--focused-invalid';
				}
			}

			return stepperInputClass;
		},
	},

	watch: {
		modelValue(value) {
			this.internalValue = value;
		},

		internalValue(value) {
			if (!value) {
				this.internalValue = 0;
			}

			if (typeof value === 'string') {
				this.internalValue = +this.internalValue;
			}

			if (value < this.min) {
				/**
				* Event that indicates that the reported value is outside the accepted range.
				* @event invalid number
				* @type {Event}
				*/
				this.internalValue = this.min;
				this.$emit('invalid-number', `'The field cannot be less than ${parseInt(this.min, 10)}.'`);
			} else if (value > this.max) {
				this.internalValue = this.max;
				this.$emit('invalid-number', `'The field cannot be greater than ${parseInt(this.max, 10)}.'`);
			} else {
				/**
				* Event used to implement the v-model.
				* @event input
				* @type {Event}
				*/
				this.$emit('update:modelValue', parseInt(value, 10));
			}
		},
	},

	methods: {

		changeValue(amount) {
			this.internalValue = parseInt(this.internalValue) + amount;
		},
		startLongPress(amount) {
			if (this.disabled) return;
			
			this.changeValue(amount);
			
			this.longPressTimer = setTimeout(() => {
				this.longPressInterval = setInterval(() => {
					this.changeValue(amount);
				}, 50);
			}, 400);
		},
		stopLongPress() {
			clearTimeout(this.longPressTimer);
			clearInterval(this.longPressInterval);
		},
	},
};
</script>
<style lang="scss" scoped>
@import '../assets/sass/tokens.scss';

.label {
	&__required-indicator {
		color: $rc-600;
	}

	&__content {
		margin: mb(1);
	}
}

.stepper-input {
	display: flex;
	outline: 1px solid $n-50;
	border-radius: 6px;
	height: 40px;
	width: fit-content;
	width: -moz-fit-content;

	&__label {
		@include caption;
		font-weight: $font-weight-semibold;
		margin: mb(1);
		display: flex;
		color: $n-700;
	}

	&__icon-container {
		background-color: $n-20;
		display: flex;
		flex-direction: column;
		justify-content: center;
		border-radius: 0px 8px 8px 0px;
	}

	&__field {
		padding: pa(2);
		margin: mr(2);
		border-radius: 6px;
		border: none;
		text-align: end;
		color: $n-600;

		&--thin {
			width: 72px;
		}

		&--default {
			width: 144px;
		}

		&--wide {
			width: 284px;
		}

		&:focus {
			outline: 0;
		}
	}

	&--focused {
		@extend .stepper-input;
		outline: 1px solid $bn-300;
		box-shadow: 0 0 0 0.2rem rgba($bn-300, .45);
	}

	&--valid {
		@extend .stepper-input;
		outline: 1px solid $gp-500;
	}

	&--invalid {
		@extend .stepper-input;
		outline: 1px solid $rc-600;
	}

	&--focused-valid {
		@extend .stepper-input--valid;
		box-shadow: 0 0 0 0.2rem rgba($gp-300, .45);
	}

	&--focused-invalid {
		@extend .stepper-input--invalid;
		box-shadow: 0 0 0 0.2rem rgba($rc-300, .45);
	}

	&--disabled {
		background-color: $n-10;
		pointer-events: none;
	}

	&__icon--plus {
		background: none;
		border: none;
		padding: 0;
		outline: none;
		padding: pt(1);
		display: flex;
		color: $n-600;
		transition: all 0.2s ease-out;
		height: 50%;
		cursor: pointer;

		&:hover {
			background-color: $bn-400;
			color: $n-0;
			border-radius: 0px 8px 0px 0px;
		}

		&:active {
			background-color: $bn-500;
			border-radius: 0px 8px 0px 0px;
		}
	}

	&__icon--minus {
		background: none;
		border: none;
		padding: 0;
		outline: none;
		padding: pt(1);
		display: flex;
		color: $n-600;
		transition: all 0.2s ease-out;
		height: 50%;
		cursor: pointer;

		&:hover {
			background-color: $bn-400;
			color: $n-0;
			border-radius: 0px 0px 8px 0px;
		}

		&:active {
			background-color: $bn-500;
			border-radius: 0px 0px 8px 0px;
		}
	}

	&__error-message {
		@include caption;
		color: $rc-600;
		margin: mt(1);
	}
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
	-webkit-appearance: none;
	margin: ma(0);
}

input[type=number] {
	-moz-appearance: textfield;
}

input[type=number]{
	// width: 68px;
}

input:disabled {
	background: none !important;
}
</style>
