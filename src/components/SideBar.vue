<template>
	<div
		:class="mainClass"
	>
		<div :class="`variant-resolver--${variant}`">
			<div class="side-bar__header">
				<div class="side-bar__logo">
					<slot
						v-if="!collapsed"
						name="logo"
					>
						<img :src="logoImage">
					</slot>
				</div>

				<div
					v-if="collapsible"
					v-rdstip="collapsedTooltipClass"
					class="side-bar__collapsible"
					@click="handleCollapse"
				>
					<rds-icon
						v-if="!collapsed"
						name="sidebar-collapse-outline"
					/>

					<rds-icon
						v-else
						name="sidebar-expand-outline"
					/>
				</div>
			</div>

			<ul
				:class="{'side-bar__container': items.length >= 1}"
			>
				<li
					v-for="(item, index) in items"
					:key="`${index}-${item.name}-item`"
					role="presentation"
					@mouseleave="itemsWithVisibilityController[index].show = false"
				>
					<div
						:id="item.label"
						v-rdstip="(collapsed && item.type === 'link') ? item.label : null"
						class="side-bar__item-container"
						:class="isActive(item) ? 'side-bar__item-container--active' : 'side-bar__item-container--inactive'"
						@click="(event) => handleClick(event, item)"
						@mouseover="itemsWithVisibilityController[index].show = true"
					>
						<div
							v-if="!!item.items || item.type === 'link'"
							class="side-bar__item"
							:class="isActive(item) ? 'side-bar__item--active' : 'side-bar__item--inactive'"
						>
							<div>
								<rds-icon
									:name="item.icon"
									width="20"
									height="20"
								/>
								<span>{{ item.label }}</span>
							</div>

							<rds-icon
								v-if="!!item.items && item.items.length > 0"
								:key="expandItemControl"
								:name="resolveCollapsibleItemIcon(item)"
								:class="{'item__caret': isActive(item)}"
								width="16"
								height="16"
							/>
						</div>

						<router-link
							v-else
							v-rdstip="collapsed ? item.label : null"
							:to="routerPushTo(item)"
							class="side-bar__item"
							:class="isActive(item) ? 'side-bar__item--active' : 'side-bar__item--inactive'"
							@click="(event) => handleClick(event, item)"
						>
							<div>
								<rds-icon
									:name="item.icon"
									width="20"
									height="20"
								/>
								<span>{{ item.label }}</span>
							</div>
						</router-link>
					</div>

					<Transition
						v-if="!collapsed"
						name="collapse"
					>
						<div
							v-show="resolveItemCollapse(item)"
							class="side-bar__subitem-container"
						>
							<div
								class="side-bar__subitems"
							>
								<a
									v-for="(subitem, idx) in item.items"
									:key="`${idx}-${subitem.name}-item`"
									class="side-bar__subitem"
									:class="isActive(subitem) && (subitem?.type !== 'external') ? 'side-bar__subitem--active' : 'side-bar__subitem--inactive'"
									:href="subitem?.type === 'external' ? subitem.route.path : 'javascript:void(0)'"
									:target="subitem?.type === 'external' ? '_blank' : '_self'"
									rel="noopener noreferrer"
								>
									<div
										v-if="subitem?.type === 'external'"
										class="side-bar__subitem-link"
									>
										{{ subitem.label }}

										<rds-icon
											height="16"
											width="16"
											name="open-in-new-tab-outline"
										/>
									</div>
									<router-link
										v-else
										class="side-bar__subitem-link"
										:to="routerPushTo(subitem)"
									>
										{{ subitem.label }}
									</router-link>
								</a>
							</div>
						</div>
					</Transition>
					<div v-if="item.items && collapsed">
						<rds-rich-tooltip
							v-model="itemsWithVisibilityController[index].show"
							:target-id="item.label"
							default-placement="bottom-start"
							width="160"
						>
							<div
								class="side-bar__subitems--collapsed"
							>
								<p class="side-bar__tooltip-title">
									{{ item.label }}
								</p>
								<a
									v-for="(subitem, idx) in item.items"
									:key="`${idx}-${subitem.name}-item`"
									class="side-bar__subitem"
									:href="subitem?.type === 'external' ? subitem.route.path : 'javascript:void(0)'"
									target="_blank"
									rel="noopener noreferrer"
									@click="(event) => handleClick(event, subitem)"
								>
									<div
										v-if="subitem?.type === 'external'"
										class="side-bar__subitem-link"
									>
										{{ subitem.label }}
										<rds-icon
											height="16"
											width="16"
											name="open-in-new-tab-outline"
										/>
									</div>
									<router-link
										v-else
										class="side-bar__subitem-link"
										:to="routerPushTo(subitem)"
									>
										{{ subitem.label }}
									</router-link>
								</a>
							</div>
						</rds-rich-tooltip>
					</div>
				</li>
			</ul>
		</div>

		<div
			:class="`side-bar__footer variant-resolver--${variant}`"
		>
			<div
				id="menu-profile"
				class="side-bar__avatar"
				@click.stop="showPopover = !showPopover"
			>
				<rds-avatar
					:src="userPicture"
					:name="userName"
					variant="white"
					size="lg"
				/>

				<Transition
					name="fade"
				>
					<div
						v-if="!collapsed"
					>
						<p>{{ userName }}</p>
						<p>{{ userRole }}</p>
					</div>
				</Transition>
			</div>

			<ul>
				<li
					v-if="showLogout"
					v-rdstip="logoutTooltipText"
					class="side-bar__logout-button"
					@click="$emit('logout', true)"
				>
					<rds-icon
						name="logout-outline"
						width="20"
						height="20"
					/>
				</li>
			</ul>
			<div v-if="shouldShowProfileMenu">
				<rds-popover
					v-model="showPopover"
					right-aligned
					target-id="menu-profile"
					width="160"
					height="160"
				>
					<div
						v-on-click-outside="hide"
						class="dropdown-button__dropdown"
					>
						<div
							v-for="(item, index) in popoverDropdownItems"
							:key="index"
						>
							<div
								class="dropdown__container"
								@click="handleProfileMenuOptionClick(item)"
							>
								<rds-icon
									class="dropdown__icon"
									height="22"
									width="22"
									:name="item.icon"
								/>
								<span class="dropdown__text">{{ item.name }}</span>
							</div>
						</div>
						<div>
							<div
								class="dropdown__container"
								@click="$emit('logout', true)"
							>
								<rds-icon
									name="logout-outline"
									width="20"
									height="20"
								/>
								<span class="dropdown__text">Logut</span>
							</div>
						</div>
					</div>
				</rds-popover>
			</div>
		</div>
	</div>
</template>

<script>
import isEqual from 'lodash.isequal';
import isEmpty from 'lodash.isempty';
import RdsIcon from './Icon.vue';
import RdsAvatar from './Avatar.vue';
import RdsRichTooltip from './RichTooltip.vue';
import RdsPopover from './Popover.vue';
import Rdstip from '../utils/directives/rdstip';

import { colorOptions, colorHexCode } from '../utils/constants/colors';

export default {
	directives: {
		rdstip: Rdstip,
	},
	components: {
		RdsIcon,
		RdsAvatar,
		RdsRichTooltip,
		RdsPopover,
	},

	props: {
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
		 * Defines the list of SideBar items to be
		 * shown. The list items must be
		 * objects with path or route, and with a label
		 */
		items: {
			type: Array,
			default: () => ([]),
			required: true,
			validator: (values) => {
				const invalidValues = values.filter((value) => {
					const hasNotRoute = isEmpty(value.path) && isEmpty(value.route);
					const hasInvalidItems = isEmpty(value.items)
						|| value.items.filter(item => (isEmpty(item.path) && isEmpty(item.route))).length;
					return isEmpty(value.label) || (hasInvalidItems && hasNotRoute);
				});
				return !invalidValues.length;
			},
		},
		/**
		* The active SideBar item
		*/
		activeItem: {
			type: Object,
			default: () => ({}),
			required: true,
		},
		/**
		* Controls menu/dropdown display when clicking on profile information
		*/
		showProfileMenu: {
			type: Boolean,
			default: false,
		},
		/**
		* Controls the menu/dropdown items displayed when clicking on profile information
		*/
		profileMenuItems: {
			type: Array,
			default: () => ([]),
		},
		/**
		* Controls display of the "exit" button in the sidebar footer
		*/
		showLogout: {
			type: Boolean,
			default: true,
		},
		/**
		* Name of the logged in user. This information is placed next to the Avatar
		*/
		userName: {
			type: String,
			default: '',
		},
		/**
		* Logged in user profile. This information is placed next to the Avatar
		*/
		userRole: {
			type: String,
			default: '',
		},
		/**
		* Image of the logged in user. Information is used to assemble the Avatar
		*/
		userPicture: {
			type: String,
			default: null,
		},
		/**
		 * Allows the sidebar to be collapsed into a minimal version
		*/
		collapsible: {
			type: Boolean,
			default: false,
		},
		/**
		 * Image of the logo that will be rendered
		*/
		logoImage: {
			type: String,
			default: null,
		},
		/**
		 * Activates sidebar light mode
		*/
		light: {
			type: Boolean,
			default: false,
		},
		/**
		* Controls the state of the sidebar, whether open or collapsed.
		*/
		collapsibleState: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			internalActiveItem: {},
			collapsed: false,
			showUncollapsedItems: true,
			colorOptions,
			expandItemControl: 0,
			itemsWithVisibilityController: [],
			logoutTooltipText: 'Logout',
			showPopover: false,
		};
	},

	computed: {
		mainClass() {
			if (this.light) {
				if (this.collapsed) {
					return 'side-bar--light--collapsed';
				}

				return 'side-bar--light';
			}

			if (this.collapsed) {
				return 'side-bar--dark--collapsed';
			}

			return 'side-bar--dark';
		},
		collapsedTooltipClass() {
			return this.collapsed ? 'Maximize' : 'Minimize';
		},
		shouldShowProfileMenu() {
			return this.showProfileMenu && this.profileMenuItems?.length > 0;
		},
		avatarCursorResolver() {
			return this.shouldShowProfileMenu ? 'pointer' : 'default';
		},
	},

	watch: {
		items: {
			handler(newValue) {
				const filtered = newValue.filter(item => item.name === this.activeItem.name);
				[this.internalActiveItem] = filtered.length ? filtered : newValue;
			},
			immediate: true,
		},

		collapsibleState: {
			handler(newValue) {
				this.collapsed = newValue;
			},
			immediate: true,
		},

		activeItem: {
			handler(newValue) {
				this.internalActiveItem = newValue;
			},
			immediate: true,
		},

		collapsed(newValue) {
			if (newValue) {
				this.showUncollapsedItems = false;
				return;
			}

			setTimeout(() => {
				this.showUncollapsedItems = true;
			}, 500);
		},
	},

	created() {
		this.internalActiveItem = this.activeItem;
		this.collapsed = this.collapsibleState;
		
		this.items.forEach((item, idx) => {
			this.itemsWithVisibilityController.push({
				name: item.label,
				index: idx,
				show: false,
			})
		});
	},

	methods: {
		colorHexCode,

		handleClick(event, item) {
			if (!isEqual(this.internalActiveItem, item)) {
				this.internalActiveItem = item;
				this.showUncollapsedItems = true;
				this.expandItemControl += 1;
				return;
			}

			if (!!item.items && item.items.length > 0) {
				this.showUncollapsedItems = !this.showUncollapsedItems;
				this.expandItemControl += 1;
				return;
			}

			if (item.icon) {
				this.showUncollapsedItems = false;
				this.expandItemControl += 1;
			}

			/**
			 * Event emitted when one of the SideBar items is clicked
			* @event sidebar-click
			* @type {Event}
			*/
			this.$emit('sidebar-click', this.internalActiveItem);
		},

		isActive(item) {
			let hasActiveSubitem = false;
			let hasActiveItem = false;

			if (!!item.items && item.items.length > 0) {
				hasActiveSubitem = item.items.some(item => {
					return isEqual(item, this.internalActiveItem)
				})
			}

			hasActiveItem = isEqual(this.internalActiveItem, item);
			return hasActiveSubitem || hasActiveItem;
		},

		resolveRoute({ route, path }) {
			const to = isEmpty(route) ? path : route;
			return to instanceof String ? { path: to } : to;
		},

		routerPushTo(item) {
			return this.resolveRoute(item) ? this.resolveRoute(item).path : null;
		},

		handleCollapse() {
			this.$emit('collapse-click', !this.collapsed);
			this.collapsed = !this.collapsed;
		},

		resolveItemCollapse(item) {
			return (!!item.items && item.items.length > 0)
				&& this.isActive(item)
				&& this.showUncollapsedItems;
		},

		resolveCollapsibleItemIcon(item) {
			return this.resolveItemCollapse(item)
				? 'caret-up-outline'
				: 'caret-down-outline';
		},

		handleProfileMenuOptionClick (actionName) {
			/**
			 * Event emitted when one of the profile dropdown menu items is clicked
			* @event profile-menu-option-click
			* @type {Event}
			*/
			this.$emit('profile-menu-option-click', actionName);
			this.showPopover = false;
		}
	},
};
</script>

<style lang="scss">
@import '../assets/sass/tokens.scss';
.fade-enter-active {
	transition: opacity 0.5s ease;
}

.fade-enter-from {
	opacity: 0;
}

.collapse-enter-active,
.collapse-leave-active {
	transition: all 0.5s ease;
}

.collapse-enter-from,
.collapse-leave-to {
	transition: all 0.35s ease;
	opacity: 0;
	transform: translateY(-8px);
}

.side-bar--dark {
	background: linear-gradient(223deg, #4B545B 18.22%, #31393f 100%);
	box-shadow: 0px 3.8957247734069824px 7.791449546813965px 0px rgba(16, 24, 64, 0.04);
	display: flex;
	flex-direction: column;
	height: 100vh;
	justify-content: space-between;
	padding: pTRBL(8, 3, 8, 3);
	width: 244px;
	transition: width 0.5s;

	.side-bar {
		&__tooltip-title {
			font-size: 16px;
			font-weight: 600;
			margin: my(2);
		}

		&__subitem {
			color: $n-100;
			cursor: pointer;
			padding: py(1);
			@include caption;

			&--active {
				color: $n-0;
				font-weight: 700;
			}

			&:hover {
				color: $n-0;
			}

			&-container {
				padding: pl(7);
				margin: mt(2);
				transition: all 0.35s ease;
			}

			&-link {
				width: 100%;
				display: block;
				display: flex;
				align-items: center;
				gap: spacer(2);
			}

			&--collapsed {
				display: flex;
				flex-direction: column;
				gap: spacer(2);
				margin: ml(2);
			}
		}


		&__subitems {
			border-left: 1px solid $n-300;
			display: flex;
			flex-direction: column;
			gap: spacer(4);
			padding: pTRBL(1, 0, 1, 4);
		}

		&__header {
			display: flex;
			align-items: center;
			justify-content: center;
			gap: spacer(4);
			margin-bottom: 40px;
		}

		&__logo {
			display: flex;
			align-items: center;
			max-width: 144px;
		}

		&__logo > img {
			width: 100%;
		}

		&__container {
			display: flex;
			flex-direction: column;
			gap: spacer(3);
			list-style: none;
			transition: all 0.9s ease;
			padding: pa(0);
		}

		&__item-container {
			cursor: pointer !important;

			&--active {
				background-color: $appbar-color;
				border: 1px solid rgba(100, 115, 130, 0.50);
				border-radius: $border-radius-extra-small;
				color: $n-10;
				cursor: default;
				height: fit-content;
				padding: pTRBL(3, 4, 3, 4);
				transition: $interaction;
				width: 100%;
			}

			&--inactive {
				background-color: transparent;
				border: 1px solid transparent;
				border-radius: $border-radius-extra-small;
				padding: pTRBL(3, 4, 3, 4);
				transition: $opening;
			}
		}

		&__item > div{
			align-items: center;
			display: flex;
			gap: spacer(2);
		}

		&__item span {
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}

		&__item {
			@include caption;
			align-items: center;
			cursor: pointer;
			display: flex;
			justify-content: space-between;
			overflow: hidden;
			text-overflow: ellipsis;
			text-align: center;
			text-decoration: none !important;

			&--active {
				color: $n-0;
				font-weight: 700;
			}

			&--active:hover {
				color: $n-0;
			}

			&--inactive {
				color: $n-10;
				height: fit-content;
			}

			&--inactive:hover {
				color: $n-0;
			}
		}

		&__logout-button {
			padding: pa(3);
			border-radius: $border-radius-small;
		}

		&__logout-button:hover {
			color: $n-0;
			background-color: #576169;
			outline: 1px solid rgba(#647382, 0.5);
			cursor: pointer;
		}

		&__footer {
			display: flex;
			align-items: center;
			justify-content: space-between;
			gap: spacer(2);
			padding: pTRBL(2, 1, 2, 1);

			& > ul {
				margin: 0;
				padding: 0;

				& > li {
					align-items: center;
					color: $n-0;
					display: flex;
					gap: spacer(2);
				}
			}
		}

		&__avatar {
			align-items: center;
			display: flex;
			gap: spacer(3);
			cursor: v-bind(avatarCursorResolver);
		}

		&__avatar > div > p:nth-child(1) {
			@include body-2;
			color: $n-0;
			font-weight: $font-weight-bold;
			margin: mb(0);
		}

		&__avatar > div > p:nth-child(2) {
			@include caption;
			color: $n-0;
			margin: 0;
		}
		/* Necessary so that the avatar has the cursor pointer without it being
			The component's "native" popover action is displayed when the clickable prop is passed to it */
		&__avatar > #avatar-dropdown {
			cursor: v-bind(avatarCursorResolver)!important;
		}

		&__collapsible {
			padding: pa(3);
			cursor: pointer;
			border-radius: $border-radius-extra-small;
			width: 42px;
			height: 42px;
			color: $n-0;

			&:hover {
				background-color: #576169;
				outline: 1px solid rgba(#647382, 0.5);
			}

			svg {
				width: 18px;
				height: 18px;
			}
		}
	}
}

.side-bar--light {
	@extend .side-bar--dark;

	.variant-resolver {
		@include variantResolver using ($color-name, $shade-50, $shade-100, $shade-200, $shade-300, $base-color, $shade-500, $shade-600) {
			--system-background-variant: #{$shade-50};
			
			@if ($color-name != 'gray') {
				--system-border-variant: #{$shade-200};
				--system-text-variant: #{$base-color};
			} @else {
				--system-border-variant: #{$shade-600};
				--system-text-variant: #{$n-800};
			}

		}
	}

	background: $n-0;
	border-right: 1px solid $n-30;
	width: 245px;

	.side-bar {
		&__subitem {
			color: $n-700;

			&--active {
				color: var(--system-text-variant);
			}

			&:hover {
				color: var(--system-text-variant);
			}
		}

		&__item {
			&--active {
				color: var(--system-text-variant);
			}

			&--active:hover {
				color: var(--system-text-variant);
			}

			&--inactive {
				color: $n-700;
			}

			&--inactive:hover {
				color: $n-700;
			}
		}

		&__item-container {
			&--active {
				background-color: var(--system-background-variant);
				border: 1px solid var(--system-border-variant);
			}
		}

		&__avatar > div > p:nth-child(1) {
			color: $n-700;
			cursor: pointer;
		}

		&__avatar > div > p:nth-child(2) {
			color: $n-700;
			cursor: pointer;
		}

		&__footer {
			& > ul {
				& > li {
					color: $n-700;
				}
			}
		}

		&__logout-button:hover {
			color: var(--system-text-variant);
			background-color: var(--system-background-variant);
			outline: 1px solid var(--system-border-variant);
		}

		&__collapsible {
			color: $n-700;

			&:hover {
				background-color: var(--system-background-variant);
				outline: 1px solid var(--system-border-variant);
				color: var(--system-text-variant);
			}
		}
	}
}

.side-bar--dark--collapsed {
	@extend .side-bar--dark;

	width: 68px;
	transition: width 0.5s;

	.side-bar {
		&__header {
			flex-direction: column;
			gap: 0;
		}

		&__item-container {
			&--active {
				padding: pa(3);
				transition: padding 0s;
			}

			&--inactive {
				padding: pa(3);
				transition: padding 0s;
			}
		}

		&__footer {
			flex-direction: column;
			gap: spacer(4);
		}

		&__logo {
			max-width: 44px;
		}
	}
}

.side-bar--light--collapsed {
	@extend .side-bar--dark--collapsed;
	@extend .side-bar--light;

	.side-bar__subitem {
		color: $n-100;
		&:hover {
			color: $n-0;
		}
	}

	width: 69px;
}

.item {
	.caret {
		transform: rotate(-180deg);
		transition: $opening;
	}
}

.dropdown {
	&__container {
		display: flex;
		gap: 12px;
		align-items: center;
		cursor: pointer;
		border-radius: 6px;
		padding: pa(2);
		&:hover {
			background-color: $n-20;
		}
	}
	&__text {
		color: $n-600;
		@include body-2;
		padding: py(1)
	}
	&__icon {
		color: $n-600;
	}
}

.popover {
	display: flex;
	align-items: center;
}
</style>