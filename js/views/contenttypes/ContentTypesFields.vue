<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div class="has-text-centered mb-lg">
			<p class="is-size-4">{{ resource.name }}</p>
		</div>

		<tabs class="is-marginless" :tabs="[
			{route: 'contenttypes.edit', label: 'hierarchy::contenttypes.single', active: false},
			{route: 'contenttypes.fields', label: 'hierarchy::contentfields.multiple', active: true}
		]"></tabs>

		<div class="paper">
			<div class="paper__body paper__body--noside">
				<div class="paper__main">
					<div class="relation-field">
						<draggable v-if="$can('write_contenttypes')" :tag="'ul'" v-model="fields" @end="saveSortedFields" :animate="200" handle=".handle">
							<li class="related-item related-item--handled" v-for="(field, i) in fields" :key="field.id">
								<span class="handle icon is-medium has-color-blue">
									<i class="fas fa-grip-lines"></i>
								</span>
								<router-link :to="{ name: 'contentfields.edit', params: {parent: field.content_type_id, id: field.id} }">{{ field.label }}</router-link> <span class="has-color-grey-darker">[{{ trans.get('hierarchy::contentfields.' + field.type) }}]</span>
								<div class="dropdown is-hoverable is-right" v-if="$can('write_contenttypes')">
									<div class="dropdown-trigger">
										<button class="button is-compact is-borderless" aria-haspopup="true">
											<i class="icon fas fa-ellipsis-v"></i>
										</button>
									</div>
									<div class="dropdown-menu" role="menu">
										<div class="dropdown-content">
											<div class="dropdown-label" v-text="trans.get('foundation::general.options')"></div>
											<router-link :to="{ name: 'contentfields.edit', params: {parent: field.content_type_id, id: field.id} }" class="dropdown-item">
												<i class="icon fas fa-edit has-color-grey-darker"></i> {{ trans.get('hierarchy::contentfields.edit') }}
											</router-link>
											<a href="#" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal({ bulk: false, route: 'contenttypes/' + field.content_type_id + '/fields/' + field.id })">
												<i class="icon fas fa-trash"></i> {{ trans.get('hierarchy::contentfields.delete') }}
											</a>
										</div>
									</div>
								</div>
							</li>
						</draggable>
						<ul v-else>
							<li class="related-item related-item--handled" v-for="(field, i) in fields" :key="field.id">
								<router-link :to="{ name: 'contentfields.edit', params: {parent: field.content_type_id, id: field.id} }">{{ field.label }}</router-link> <span class="has-color-grey-darker">[{{ trans.get('hierarchy::contentfields.' + field.type) }}]</span>
							</li>
						</ul>
					</div>
				</div>
			</div>
			<div class="paper__footer" v-if="$can('write_contenttypes') && resource.id">
				<div class="field has-addons is-pulled-right">
					<div class="control">
						<router-link class="button icon-only-wide is-primary" :to="{ name: 'contentfields.create', params: {id: resource.id}}">
							<i class="icon fas fa-plus"></i>
						</router-link>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import {View, Shower, Tabs, Datable, DatableDropdown, format_date, RequiresPermissions} from 'umomega-foundation'
import draggable from 'vuedraggable'
import {assess_error} from 'umomega-foundation'

export default {
	mixins: [ View, Shower, RequiresPermissions ],
	components: { Datable, DatableDropdown, Tabs, draggable },
	data() { return {
		titleLabel: 'hierarchy::contentfields.multiple',
		breadcrumbs: [
			{to: { name: 'contenttypes.index'}, text: this.$root.trans.get('hierarchy::contenttypes.multiple')}
		],
		showRoute: 'contenttypes',
		guardedBy: 'read_contenttypes',
		fields: []
	}},
	created() {
		this.load()

		var self = this

		Event.$off('resources-deleted')
		Event.$on('resources-deleted', function(data) {
			self.load()
		})
	},
	methods: {
		load() {
			var self = this;

			axios.get(api_url_with_token('contenttypes/' + self.$route.params.id  + '/fields'))
				.then(function(response) {
					self.fields = response.data;
					self.isLoaded = true
				})
				.catch(function(error) { assess_error(error) })
		},
		saveSortedFields() {
			let sorted = this.fields.map(function(f) {
				return f.id
			})
			let self = this

			axios.put(api_url_with_token('contenttypes/' + self.$route.params.id + '/fields/sort'), {sorted: sorted})
				.catch(function(error) { assess_error(error) })
		},
		openDeleteModal(payload) {
			Event.$emit('delete-modal-open', payload)
		}
	},

}
</script>