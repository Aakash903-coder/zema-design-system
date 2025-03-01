<template>
	<div :class="resolveMainClasses">
		<div :class="`variant-resolver--${variant}`">
			<div class="mobile-navigation__header">
				<div class="mobile-navigation__title">
					{{ internalActiveItem.label || '' }}
				</div>

				<div
					class="mobile-navigation__menu-btn"
					@click="handleOpenSidebar"
				>
					<rds-icon
						name="menu-outline"
					/>
				</div>
			</div>

			<div
				class="mobile-navigation__sidebar"
				:class="{
					'mobile-navigation__sidebar--expanded': openSidebar
				}"
			>
				<div class="mobile-navigation__sidebar-header">
					<div class="mobile-navigation__sidebar-logo">
						<!-- @slot Sidebar logo rendering slot.-->
						<slot name="sidebar-logo">
							<img :src="sidebarLogo">
						</slot>
					</div>

					<div
						class="mobile-navigation__sidebar-btn"
						@click="handleCloseSidebar"
					>
						<rds-icon
							name="x-outline"
						/>
					</div>
				</div>

				<div class="mobile-navigation__sidebar-content">
					<div class="mobile-navigation__sidebar-items">
						<router-link
							v-for="item in items"
							:key="item.label"
							:to="routerPushTo(item)"
							class="mobile-navigation__sidebar-item"
							:class="{
								'mobile-navigation__sidebar-item--active': isActive(item),
							}"
							@click="handleItemClick(item)"
						>
							<rds-icon
								:name="item.icon"
								width="24"
								height="24"
							/>

							<span>{{ item.label }}</span>
						</router-link>
					</div>

					<div class="mobile-navigation__sidebar-footer">
						<div
							class="mobile-navigation__sidebar-user-info"
							@click="emit('profile-click')"
						>
							<rds-avatar
								:src="user.picture || ''"
								:name="user.name || ''"
								:variant="variant"
								size="lg"
							/>

							<div class="mobile-navigation__sidebar-user-text">
								<p>{{ user.name || '' }}</p>
								<p>{{ user.role || '' }}</p>
							</div>
						</div>

						<button
							class="mobile-navigation__sidebar-logout"
							@click="emit('logout')"
						>
							<rds-icon
								name="logout-outline"
								width="24"
								height="24"
							/>
						</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, defineProps, defineEmits, computed, watch } from 'vue';
import { isEqual, isEmpty } from 'lodash';
import RdsIcon from './Icon.vue';
import RdsAvatar from './Avatar.vue';

const props = defineProps({
	/**
	 * The color variant. There are 10 variants implemented: 'green', 'teal',
	 * 'blue', 'indigo', 'violet', 'pink', 'red', 'orange', 'amber' and 'white'.
	 * The variant will only take effect when the SideBar is in light mode.
	 */
	variant: {
		type: String,
		default: 'green',
	},
	/**
	 * Activates navigation light mode.
	*/
	light: {
		type: Boolean,
		default: false,
	},
	/**
	 * Defines the logo that will appear in the main navigation container
	*/
	// mainLogo: {
	// 	type: String,
	// 	default: '',
	// },
	/**
	 * Defines the logo that will appear in the navigation sidebar
	*/
	sidebarLogo: {
		type: String,
		default: '',
	},
	/**
	 * Defines the list of navigation items to be
	 * shown. The list items must be
	 * objects with path or route, and with a label.
	 */
	items: {
		type: Array,
		default: () => [],
	},
	/**
	* The active navigation item.
	*/
	activeItem: {
		type: Object,
		default: () => {},
	},
	/**
	 * Defines information relating to the user. The object must follow the signature:
	 * { name: String, role: String, picture: String }
	*/
	user: {
		type: Object,
		default: () => {},
	},
	/**
	 * Ddefines whether the component will be in sticky mode
	*/
	sticky: {
		type: Boolean,
		default: false,
	},
});

const emit = defineEmits([
	/**
	 * Event emitted when logout button is clicked
	* @event logout
	* @type {Event}
	*/
	'logout',
	/**
	 * Event emitted when one of the navigation items is clicked
	* @event item-click
	* @type {Event}
	*/
	'item-click',
	/**
	 * Event emitted when click is made on user information
	* @event profile-click
	* @type {Event}
	*/
	'profile-click',
]);

const internalActiveItem = ref(props.activeItem);
const openSidebar = ref(false);

const resolveMode = computed(() => {
	return (props.light) ? 'light' : 'dark';
});

const resolveMainClasses = computed(() => {
	let classes = `mobile-navigation mobile-navigation--${resolveMode.value}`;

	if (props.sticky) {
		classes += ' mobile-navigation--sticky';
	}

	return classes;
});

watch(openSidebar, async (newValue, oldValue) => {
	if (newValue !== oldValue) {
		mustDisableExternalScrolls(newValue);
	}
});

watch(() => props.activeItem, (newValue) => {
	internalActiveItem.value = newValue;
});

const handleOpenSidebar = () => {
	openSidebar.value = true;
};

const handleCloseSidebar = () => {
	openSidebar.value = false;
};

const isActive = (item) => isEqual(item, internalActiveItem.value);

const handleItemClick = (item) => {
	if (isEmpty(props.activeItem)) {
		internalActiveItem.value = item;
	}

	openSidebar.value = false;

	emit('item-click', item);
};

const resolveRoute = ({ route, path }) => {
	const to = isEmpty(route) ? path : route;
	return to instanceof String ? { path: to } : to;
};

const routerPushTo = (item) => {
	return resolveRoute(item) ? resolveRoute(item).path : null;
};

const mustDisableExternalScrolls = (value) => {
	document.body.style.overflow = value ? 'hidden' : 'auto';
};
</script>

<style lang="scss" scoped>
@import '../assets/sass/tokens.scss';

.mobile-navigation {
	top: 0;
	left: 0;
	width: 100%;
	z-index: 1000;
	box-shadow: $shadow-sm;

	&--sticky {
		position: sticky;
	}

	&__header {
		padding: pTRBL(5, 4, 5, 4);
		background-color: rgba(#FFF, .87);
		backdrop-filter: blur(5px);
	}

	&__title {
		@include subheading-1;
		margin: mTRBL(0, 0, 0, 10);
	}

	&__menu-btn {
		position: absolute;
		top: spacer(6);
		left: spacer(4);
	}

	&__sidebar {
		display: flex;
		flex-direction: column;
		gap: spacer(6);
		top: 0;
		left: 0;
		position: absolute;
		padding: pYX(4, 5);
		height: 100svh;
		width: 100%;
		margin-left: calc(0px - 100%);
		transition: margin-left 0.3s ease;
	}

	&__sidebar--expanded {
		margin-left: 0;
		transition: margin-left 0.3s ease;
	}

	&__sidebar-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	&__sidebar-logo img {
		max-width: 147px;
		width: 100%;
	}

	&__sidebar-btn {
		margin-right: spacer(1);
	}

	&__sidebar-content {
		flex-grow: 1;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		margin-bottom: spacer(8);
	}

	&__sidebar-items {
		display: flex;
		flex-direction: column;
		gap: spacer(4);
	}

	&__sidebar-item {
		@include body-1;
		display: flex;
		align-items: center;
		padding: pYX(4, 5);
		border-radius: 10px;
		gap: 10px;
		background: none;
		transition: background 0.25s ease-in-out;

		&--active {
			font-weight: 700;
		}
	}

	&__sidebar-footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	&__sidebar-user-info {
		display: flex;
		align-items: center;
		gap: spacer(4);
	}

	&__sidebar-user-text {
		display: flex;
		flex-direction: column;
		gap: 0;

		& p {
			margin: 0;
		}

		& p:first-child {
			@include body-1;
			font-weight: $font-weight-bold;
		}

		& p:last-child {
			@include body-2;
		}
	}

	&__sidebar-logout {
		padding: pa(2);
		background: none;
		border: none;
		cursor: pointer;
	}
}

.mobile-navigation--dark {
	&.mobile-navigation {
		background: linear-gradient(183deg, #4B545B 26.25%, #262B2F 183.33%);
		color: $n-0;
	}

	.mobile-navigation {
		&__sidebar {
			background: linear-gradient(183deg, #4B545B 26.25%, #262B2F 183.33%);
			color: $n-0;
		}

		&__sidebar-item {
			&--active {
				background: #576169;
				color: $n-0;
				border: 1px solid #64738280;
				transition: background 0.25s ease-in-out;
			}
		}

		&__sidebar-logout {
			color: $n-0;
		}
	}
}

.mobile-navigation--light {
	.variant-resolver {
		@include variantResolver using ($color-name, $shade-50, $shade-100, $shade-200, $shade-300, $base-color, $shade-500, $shade-600) {
			--system-background-variant: #{$shade-50};
			--system-border-variant: #{$shade-200};
			--system-text-variant: #{$base-color};
		}
	}

	&.mobile-navigation {
		background: $n-0;
		color: $n-700;
	}

	.mobile-navigation {
		&__sidebar {
			background: $n-0;
			color: $n-700;
		}

		&__sidebar-item {
			&--active {
				background: var(--system-background-variant);
				color: var(--system-text-variant);
				border: 1px solid var(--system-border-variant);
			}
		}

		&__sidebar-logout {
			color: $n-700;
		}
	}
}
</style>
