<!-- eslint-disable vue/multi-word-component-names -->
<template>
	<template
		v-if="highlightedText && isAValidSubstring"
	>
		<template
			v-for="(element) in substrings"
			:key="element.id"
		>
			<mark
				v-if="element.highlighted"
				class="highlight__container"
				:class="dynamicHighlightClass"
				:style="dynamicStyle"
			>
				{{ element.string }}
			</mark>
			<template v-else>
				{{ element.string }}
			</template>
		</template>
	</template>

	<mark
		v-else
		class="highlight__container"
		:class="{ dynamicHighlightClass: isAValidSubstring }"
		:style="dynamicStyle"
	>
		<!-- @slot Slot used to specify the text that will receive the highlight. -->
		<slot />
	</mark>
</template>

<script>
export default {
	props: {
		/**
		 * The Highlight variant. There are 3 variants implemented: 'info', 'success'
		 * and 'danger'
		*/
		variant: {
			type: String,
			default: 'info',
		},
		/**
		 * Specifies whether the highlight will be static or animated.
		*/
		animated: {
			type: Boolean,
			default: false,
		},
		/**
		 * Specifies the duration of the animation.
		*/
		duration: {
			type: Number,
			default: 1,
		},
		/**
		 * The waiting time until the animation starts.
		*/
		delay: {
			type: Number,
			default: 0,
		},
		/**
		* Define uma substring específica do texto enviada por slot que deve receber o destaque (highlight).
		*/
		highlightedText: {
			type: [String, null],
			default: null,
		},
		/**
		* Informa se o highlight vai ser Case Sensitive.
		*/
		caseSensitive: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			substrings: [],
		}
	},

	computed: {
		hasSlots() {
			return !!Object.keys(this.$slots).length;
		},

		slotString() {
			return this.hasSlots ? this.$slots.default()[0].children : '';
		},

		normalizedSlotString() {
			return this.normalize(this.slotString);
		},

		normalizedHighlightText() {
			return this.normalize(this.highlightedText);
		},

		isAValidSubstring() {
			return this.normalizedSlotString.includes(this.normalizedHighlightText);
		},

		paddingResolver() {
			if (this.highlightedText) {
				return '4px 1px';
			}

			return '4px';
		},

		dynamicHighlightClass() {
			let dynamicClass = '';

			if (this.animated) {
				dynamicClass = 'highlight__container--highlighted';
			}

			switch (this.variant) {
				case 'info':
					return `${dynamicClass} highlight__container--info`;
				case 'success':
					return `${dynamicClass} highlight__container--success`;
				case 'danger':
					return `${dynamicClass} highlight__container--danger`;
				default:
					return `${dynamicClass} highlight__container--info`;
			}
		},

		dynamicStyle() {
			return {
				'--duration': `${this.duration}s`,
				'--delay': `${this.delay}s`,
			};
		},
	},

	watch: {
		highlightedText() {
			this.splitContent();
		}
	},

	mounted() {
		this.splitContent();
	},

	methods: {
		normalize(string) {
			const accentedChars = {
				'a': '[aáàâã]',
				'á': '[aáàâã]',
				'à': '[aáàâã]',
				'â': '[aáàâã]',
				'ã': '[aáàâã]',
				'e': '[eéê]',
				'é': '[eéê]',
				'ê': '[eéê]',
				'i': '[ií]',
				'í': '[ií]',
				'o': '[oóôõ]',
				'ó': '[oóôõ]',
				'ô': '[oóôõ]',
				'õ': '[oóôõ]',
				'u': '[uúü]',
				'ú': '[uúü]',
				'ü': '[uúü]',
			};

			const caseSensitiveAccentedChars = {
				'a': '[aáàâã]',
				'á': '[aáàâã]',
				'à': '[aáàâã]',
				'â': '[aáàâã]',
				'ã': '[aáàâã]',
				'e': '[eéê]',
				'é': '[eéê]',
				'ê': '[eéê]',
				'i': '[ií]',
				'í': '[ií]',
				'o': '[oóôõ]',
				'ó': '[oóôõ]',
				'ô': '[oóôõ]',
				'õ': '[oóôõ]',
				'u': '[uúü]',
				'ú': '[uúü]',
				'ü': '[uúü]',
				'A': '[AÁÀÂÃ]',
				'Á': '[AÁÀÂÃ]',
				'À': '[AÁÀÂÃ]',
				'Â': '[AÁÀÂÃ]',
				'Ã': '[AÁÀÂÃ]',
				'E': '[EÉÊ]',
				'É': '[EÉÊ]',
				'Ê': '[EÉÊ]',
				'I': '[IÍ]',
				'Í': '[IÍ]',
				'O': '[OÓÔÕ]',
				'Ó': '[OÓÔÕ]',
				'Ô': '[OÓÔÕ]',
				'Õ': '[OÓÔÕ]',
				'U': '[UÚÜ]',
				'Ú': '[UÚÜ]',
				'Ü': '[UÚÜ]',
			};

			if (this.caseSensitive) {
				return string?.replace(/[aáàâãeéêiíoóôõuúüAÁÀÂÃEÉÊIÍOÓÔÕUÚÜ]/g, match => caseSensitiveAccentedChars[match] || match);
			}

			return string?.toLowerCase().replace(/[aáàâãeéêiíoóôõuúü]/g, match => accentedChars[match] || match);
		},

		splitContent() {
			let regexFlag = this.caseSensitive ? 'g' : 'gi';
			let expression = new RegExp(`(${this.normalizedHighlightText})`, regexFlag);
			let matchResult = this.slotString.split(expression);

			this.substrings = matchResult.map((string, index) => {
				return {
					string,
					highlighted: new RegExp(`^${this.normalizedHighlightText}$`, regexFlag).test(this.caseSensitive ? string : string?.toLowerCase()),
					id: index,
				};
			});
		}
	},
};
</script>
<style lang="scss" scoped>
@import '../assets/sass/tokens.scss';

mark {
	all: unset;
}

.highlight__container {
	background-size: 200%;
	border-radius: 4px;
	padding: v-bind(paddingResolver);

	&--highlighted {
		animation: highlight var(--duration) ease-in var(--delay) backwards;
	}

	&--info {
		background-image: linear-gradient(to right, rgba($bn-300, .25) 50%, transparent 50%);
	}

	&--success {
		background-image: linear-gradient(to right, rgba($gp-300, .28) 50%, transparent 50%);
	}

	&--danger {
		background-image: linear-gradient(to right, rgba($rc-300, .22) 50%, transparent 50%);
	}
}

@keyframes highlight {
	0% {
		background-position: 100%;
	}
}
</style>
