<template lang="html">
	<div
		:class="{ maximized: maximized || exclusive, exclusive }"
		class="ffz-main-menu tw-elevation-3 tw-c-background-alt tw-c-text tw-border tw-flex tw-flex-nowrap tw-flex-column"
	>
		<header class="tw-c-background tw-full-width tw-align-items-center tw-flex tw-flex-nowrap" @dblclick="resize">
			<h3 class="ffz-i-zreknarf ffz-i-pd-1">FrankerFaceZ</h3>
			<div class="tw-flex-grow-1 tw-pd-x-2">
				<!--div class="tw-search-input">
					<label for="ffz-main-menu.search" class="hide-accessible">{{ t('main-menu.search', 'Search Settings') }}</label>
					<div class="relative">
						<div class="tw-input__icon-group">
							<div class="tw-input__icon">
								<figure class="ffz-i-search" />
							</div>
						</div>
						<input type="search" class="tw-input tw-input--icon-left" :placeholder="t('main-menu.search', 'Search Settings')" autocapitalize="off" autocorrect="off" autocomplete="off" id="ffz-main-menu.search">
					</div>
				</div-->
			</div>
			<button v-if="!exclusive" class="tw-button-icon tw-mg-x-05" @click="resize">
				<span class="tw-button-icon__icon">
					<figure :class="{'ffz-i-window-maximize': !maximized, 'ffz-i-window-restore': maximized}" />
				</span>
			</button>
			<button v-if="!exclusive" class="tw-button-icon tw-mg-x-05" @click="close">
				<span class="tw-button-icon__icon">
					<figure class="ffz-i-window-close" />
				</span>
			</button>
		</header>
		<section class="tw-border-t tw-full-height tw-full-width tw-flex tw-flex-nowrap tw-overflow-hidden">
			<nav class="ffz-vertical-nav tw-c-background-alt-2 tw-border-r tw-full-height tw-flex tw-flex-column tw-flex-shrink-0 tw-flex-nowrap">
				<header class="tw-border-b tw-pd-1">
					<profile-selector
						:context="context"
						@navigate="navigate"
					/>
				</header>
				<div class="tw-full-width tw-full-height tw-overflow-hidden tw-flex tw-flex-nowrap tw-relative">
					<simplebar classes="ffz-vertical-nav__items tw-full-width tw-flex-grow-1">
						<menu-tree
							:current-item="currentItem"
							:modal="nav"
							@change-item="changeItem"
							@navigate="navigate"
						/>
					</simplebar>
				</div>
				<footer class="tw-c-text-alt tw-border-t tw-pd-1">
					<div>
						{{ t('main-menu.version', 'Version %{version}', {version: version.toString()}) }}
					</div>
					<div class="tw-c-text-alt-2">
						{{ version.build }}
					</div>
				</footer>
			</nav>
			<simplebar classes="tw-flex-grow-1">
				<menu-page
					v-if="currentItem"
					ref="page"
					:context="context"
					:item="currentItem"
					@change-item="changeItem"
					@navigate="navigate"
				/>
			</simplebar>
		</section>
	</div>
</template>

<script>

import displace from 'displacejs';

export default {
	data() {
		return this.$vnode.data;
	},

	watch: {
		maximized() {
			this.updateDrag();
		}
	},

	created() {
		this.context.context._add_user();
	},

	destroyed() {
		this.context.context._remove_user();
	},

	mounted() {
		this.updateDrag();

		this._on_resize = this.handleResize.bind(this);
		window.addEventListener('resize', this._on_resize);
	},

	beforeDestroy() {
		this.destroyDrag();

		if ( this._on_resize ) {
			window.removeEventListener('resize', this._on_resize);
			this._on_resize = null;
		}
	},

	methods: {
		changeProfile() {
			const new_id = this.$refs.profiles.value,
				new_profile = this.context.profiles[new_id];

			if ( new_profile )
				this.context.currentProfile = new_profile;
		},

		changeItem(item) {
			if ( this.$refs.page && this.$refs.page.onBeforeChange ) {
				if ( this.$refs.page.onBeforeChange(this.currentItem, item) === false )
					return;
			}

			this.currentItem = item;
			let current = item;
			while(current = current.parent) // eslint-disable-line no-cond-assign
				current.expanded = true;
		},

		updateDrag() {
			if ( this.maximized )
				this.destroyDrag();
			else
				this.createDrag();
		},

		destroyDrag() {
			if ( this.displace ) {
				this.displace.destroy();
				this.displace = null;
			}
		},

		createDrag() {
			this.$nextTick(() => {
				if ( ! this.maximized )
					this.displace = displace(this.$el, {
						handle: this.$el.querySelector('header'),
						highlightInputs: true,
						constrain: true
					});
			})
		},

		handleResize() {
			if ( this.displace )
				this.displace.reinit();
		},

		navigate(key) {
			let item = this.nav_keys[key];
			while(item && item.page)
				item = item.parent;

			if ( ! item )
				return;

			this.changeItem(item);
		}
	}
}
</script>