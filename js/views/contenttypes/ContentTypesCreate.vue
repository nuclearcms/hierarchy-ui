<template>
	<div :class="isLoaded ? 'paper reveal is-loaded' : 'paper reveal'">
		<form method="POST" action="/api/contenttypes" @submit.prevent="requestStore('contenttypes', 'contenttypes.edit')" autocomplete="off">
			
			<div class="paper__body paper__body--noside">
				<div class="paper__main">
					<FormBody :schema="schema" v-model="form" :readonly="false"/>
				</div>
			</div>

			<SubmitFooter :config="{icon: 'plus'}" v-model="form"></SubmitFooter>

		</form>
	</div>
</template>

<script>
import {View, Storer, FormBody, Form, SubmitFooter, RequiresPermissions} from 'umomega-foundation'

export default {
	mixins: [ View, RequiresPermissions, Storer ],
	components: { FormBody, SubmitFooter },
	data() { return {
		titleLabel: 'hierarchy::contenttypes.create',
		breadcrumbs: [
			{to: { name: 'contenttypes.index'}, text: this.$root.trans.get('hierarchy::contenttypes.multiple')}
		],
		guardedBy: 'write_contenttypes',
		form: new Form({name: '', is_visible: true, hides_children: false, color: {hex: '#00249C'}, is_taggable: true, allowed_children_types: []}),
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
					name: 'is_visible',
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
					name: 'is_taggable',
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
};
</script>