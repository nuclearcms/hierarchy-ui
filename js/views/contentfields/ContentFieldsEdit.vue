<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div class="has-text-centered mb-lg">
			<p class="is-size-4">{{ resource.name }}</p>
		</div>

		<div class="paper">
			<form method="POST" action="/api/contenttypes" @submit.prevent="requestUpdateDynamic()" autocomplete="off">
				
				<div class="paper__body paper__body--noside">
					<div class="paper__main">
						<FormBody :schema="schema" v-model="form" :readonly="!$can('write_contenttypes')"/>
					</div>
				</div>

				<SubmitFooter v-if="$can('write_contenttypes')" :config="{icon: 'save'}" v-model="form">
					<div class="control">
						<router-link class="button icon-only is-warning" :to="{ name: 'contentfields.create', params: {id: resource.content_type_id} }">
							<i class="icon fas fa-plus"></i>
						</router-link>
					</div>
				</SubmitFooter>

			</form>
		</div>
	</div>
</template>

<script>
import {View, Updater, Shower, Tabs, FormBody, Form, SubmitFooter, RequiresPermissions} from 'umomega-foundation'

export default {
	mixins: [ View, Updater, Shower, RequiresPermissions ],
	components: { FormBody, SubmitFooter, Tabs },
	data() { return {
		titleLabel: 'hierarchy::contentfields.edit',
		breadcrumbs: [
			{to: { name: 'contenttypes.index'}, text: this.$root.trans.get('hierarchy::contenttypes.multiple')}
		],
		guardedBy: 'read_contenttypes',
		showRoute: 'contenttypes/' + this.$route.params.parent + '/fields',
		form: new Form({label: '', description: '', search_priority: 0, is_visible: true, rules: '', default_value: '', options: ''}),
		schema: [
			{
				type: 'TextField',
				name: 'label',
				label: this.$root.trans.get('validation.attributes.label'),
				options: {required: true},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_label')
			},
			{
				type: 'TextareaField',
				name: 'description',
				label: this.$root.trans.get('validation.attributes.description'),
				options: {}
			},
			[
				{
					type: 'NumberField',
					name: 'search_priority',
					label: this.$root.trans.get('validation.attributes.search_priority'),
					options: { required: true, min: 0, max: 100, step: 1}
				},
				{
					type: 'CheckboxField',
					name: 'is_visible',
					label: this.$root.trans.get('validation.attributes.is_visible'),
					options: { required: true }
				}
			],
			{
				type: 'TextField',
				name: 'rules',
				label: this.$root.trans.get('validation.attributes.rules'),
				options: {},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_rules')
			},
			{
				type: 'TextField',
				name: 'default_value',
				label: this.$root.trans.get('validation.attributes.default_value'),
				options: {},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_default_value')
			},
			{
				type: 'TextareaField',
				name: 'options',
				label: this.$root.trans.get('validation.attributes.options'),
				options: {},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_options')
			}
		]
	}},
	watch: {
		resource(n) {
			this.breadcrumbs.splice(1)
			this.breadcrumbs.push({to: { name: 'contenttypes.fields', params: { id: n.contentType.id }}, text: n.contentType.name})
		}
	},
	methods: {
		requestUpdateDynamic() {
			this.requestUpdate('contenttypes/' + this.resource.contentType.id + '/fields')
		}
	}
}
</script>