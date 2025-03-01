<template>
	<div class="page-layout">
		<side-bar
			:variant="variant"
			:active-item="sideBarActiveItem"
			:light="light"
			:user-name="user.name || ''"
			:user-role="user.role || ''"
			:user-picture="user.picture || null"
			:collapsible="collapsibleSideBar"
			:collapsible-state="collapsibleStateSideBar"
			:items="sideBarItems"
			:logo-image="logos.default"
			:collapsed-logo-image="logos.collapsed"
			:show-profile-menu="showProfileMenu"
			:profile-menu-items="profileMenuItems"
			@profile-menu-option-click="emit('profile-menu-option-click', $event)"
			@sidebar-click="emit('sidebar-item-click', $event)"
			@logout="emit('sidebar-logout', $event)"
			@settings="emit('sidebar-settings', $event)"
		/>

		<div class="page-layout__content">
			<secondary-navigation
				v-if="navigationItems?.length > 0"
				:light="light"
				:active-item="navigationActiveItem"
				:items="navigationItems"
				@item-click="emit('navigation-item-click', $event)"
			/>

			<page-container class="page-layout__container">
				<!-- @slot Slot used for inserting PageLayout content. -->
				<slot />
			</page-container>
		</div>
	</div>
</template>

<script setup>
import { defineProps, defineEmits, computed } from 'vue';
import SideBar from './SideBar.vue';
import SecondaryNavigation from './SecondaryNavigation.vue';
import PageContainer from './PageContainer.vue';

const props = defineProps({
	/**
	 * The color variant. There are 10 variants implemented: 'green', 'teal',
	 * 'blue', 'indigo', 'violet', 'pink', 'red', 'orange', 'amber' and 'white'.
	 * The variant will only take effect when the PageLayout is in light mode.
	 */
	variant: {
		type: String,
		default: 'green',
	},
	/**
	 * Activates PageLayout light mode.
	*/
	light: {
		type: Boolean,
		default: false,
	},
	/**
	 * Defines information relating to the user. The object must follow the signature:
	 * { name: String, role: String, picture: String }
	*/
	user: {
		type: Object,
		default: () => ({}),
	},
	/**
	 * Defines whether the sidebar can be collapsed.
	*/
	collapsibleSideBar: {
		type: Boolean,
		default: false,
	},

	showProfileMenu: {
		type: Boolean,
		default: false,
	},
	/**
	 * Defines navigation sidebar items.
	 */
	sideBarItems: {
		type: Array,
		default: () => [],
	},
	/**
	 * Defines the items to be displayed in the dropdown menu shown when clicking on "profile".
	 */
	profileMenuItems: {
		type: Array,
		default: () => ([]),
	},
	/**
	 * Defines the items in the secondary navigation bar.
	 */
	navigationItems: {
		type: Array,
		default: () => [],
	},
	/** Defines the navigation sidebar logos, collapsible is the logo
	 * that will appear when the sidebar is collapsed.
	 */
	logos: {
		type: Object,
		default: () => ({}),
	},
	/** Defines the active sideBar item */
	sideBarActiveItem: {
		type: Object,
		default: () => ({}),
	},
	/** Sets the active secondary navigation item */
	navigationActiveItem: {
		type: Object,
		default: () => ({}),
	},
	/**
	* Controls the state of the sidebar, whether open or collapsed.
	*/
	collapsibleStateSideBar: {
		type: Boolean,
		default: false,
	},
});

const emit = defineEmits([
	/**
	* Event emitted when one of the SideBar items is clicked
	* @event sidebar-item-click
	* @type {Event}
	*/
	'sidebar-item-click',

	/**
	* Event emitted when the SideBar logout icon is clicked
	* @event sidebar-logout
	* @type {Event}
	*/
	'sidebar-logout',
	'popover-item-click',

	/**
	* Event emitted when the SecondaryNavbar icon is clicked
	* @event navigation-item-click
	* @type {Event}
	*/
	'navigation-item-click'
]);

const hasNavigationItems = computed(() => props.navigationItems?.length > 0);
const maxHeightCompensation = computed(() => hasNavigationItems.value ? '54px' : '0px');
</script>

<style lang="scss" scoped>
.page-layout {
	display: flex;

	&__content {
		position: relative;
		flex-grow: 1;
	}

	&__container {
		max-height: calc(100vh - v-bind(maxHeightCompensation));
		overflow-y: auto;
	}
}
</style>
