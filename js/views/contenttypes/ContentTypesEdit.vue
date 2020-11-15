<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div class="has-text-centered mb-lg">
			<p class="is-size-4">{{ resource.name }}</p>
		</div>
		
		<tabs class="is-marginless" :tabs="[
			{route: 'contenttypes.edit', label: 'hierarchy::contenttypes.single', active: true},
			{route: 'contenttypes.fields', label: 'hierarchy::contentfields.multiple', active: false}
		]"></tabs>

		<div class="paper">
			<form method="POST" action="/api/contenttypes" @submit.prevent="requestUpdate('contenttypes')" autocomplete="off">
				
				<div class="paper__body paper__body--noside">
					<div class="paper__main">
						<FormBody :schema="schema" v-model="form" :readonly="!$can('write_contenttypes')"/>
					</div>
				</div>

				<SubmitFooter v-if="$can('write_contenttypes')" :config="{icon: 'save'}" v-model="form"></SubmitFooter>

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
		titleLabel: 'hierarchy::contenttypes.edit',
		breadcrumbs: [
			{to: { name: 'contenttypes.index'}, text: this.$root.trans.get('hierarchy::contenttypes.multiple')}
		],
		guardedBy: 'read_contenttypes',
		showRoute: 'contenttypes',
		form: new Form({name: '', visible: false, hides_children: false, color: {hex: '#00249C'}, taggable: false, allowed_children_types: []}),
		schema: [
			{
				type: 'TextField',
				name: 'name',
				label: this.$root.trans.get('validation.attributes.name'),
				options: {required: true}
			},
			[
				{
					type: 'ColorField',
					name: 'color',
					label: this.$root.trans.get('validation.attributes.color'),
					options: {required: true}
				},
				{
					type: 'CheckboxField',
					name: 'visible',
					label: this.$root.trans.get('validation.attributes.visible'),
					options: {required: true}
				},
				{
					type: 'CheckboxField',
					name: 'hides_children',
					label: this.$root.trans.get('validation.attributes.hides_children'),
					options: {required: true}
				},
				{
					type: 'CheckboxField',
					name: 'taggable',
					label: this.$root.trans.get('validation.attributes.taggable'),
					options: {required: true}
				},
			],
			{
				type: 'RelationField',
				name: 'allowed_children_types',
				label: this.$root.trans.get('validation.attributes.allowed_children_types'),
				placeholder: this.$root.trans.get('hierarchy::contenttypes.search'),
				options: {multiple: true, searchroute: 'contenttypes/search'}
			}
		]
	}}
}
</script>