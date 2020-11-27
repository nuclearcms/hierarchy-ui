<template>
	<div :class="isLoaded ? 'paper reveal is-loaded' : 'paper reveal'">
		<form method="POST" action="/api/contenttypes" @submit.prevent="requestStoreDynamic()" autocomplete="off">
			
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
import {View, Storer, Shower, FormBody, Form, SubmitFooter, RequiresPermissions} from 'umomega-foundation'

export default {
	mixins: [ View, RequiresPermissions, Shower, Storer ],
	components: { FormBody, SubmitFooter },
	data() { return {
		titleLabel: 'hierarchy::contentfields.create',
		breadcrumbs: [
			{to: { name: 'contenttypes.index'}, text: this.$root.trans.get('hierarchy::contenttypes.multiple')}
		],
		guardedBy: 'write_contenttypes',
		showRoute: 'contenttypes',
		form: new Form({name: '', label: '', type: '', description: '', search_priority: 0, is_visible: true, rules: '', default_value: '', options: ''}),
		preventPopulateForm: true,
		schema: [
			{
				type: 'TextField',
				name: 'name',
				label: this.$root.trans.get('validation.attributes.name'),
				options: {required: true},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_name')
			},
			{
				type: 'TextField',
				name: 'label',
				label: this.$root.trans.get('validation.attributes.label'),
				options: {required: true},
				hint: this.$root.trans.get('hierarchy::contentfields.hint_label')
			},
			{
				type: 'SelectField',
				name: 'type',
				label: this.$root.trans.get('validation.attributes.type'),
				options: {
					required: true,
					choices: [
						{ value: 'TextField', label: this.$root.trans.get('hierarchy::contentfields.TextField') },
						{ value: 'TextareaField', label: this.$root.trans.get('hierarchy::contentfields.TextareaField') },
						{ value: 'TextEditorField', label: this.$root.trans.get('hierarchy::contentfields.TextEditorField') },
						{ value: 'MediaField', label: this.$root.trans.get('hierarchy::contentfields.MediaField') },
						{ value: 'ContentRelationField', label: this.$root.trans.get('hierarchy::contentfields.ContentRelationField') },
						{ value: 'DatetimeField', label: this.$root.trans.get('hierarchy::contentfields.DatetimeField') },
						{ value: 'CheckboxField', label: this.$root.trans.get('hierarchy::contentfields.CheckboxField') },
						{ value: 'SelectField', label: this.$root.trans.get('hierarchy::contentfields.SelectField') },
						{ value: 'NumberField', label: this.$root.trans.get('hierarchy::contentfields.NumberField') },
						{ value: 'ColorField', label: this.$root.trans.get('hierarchy::contentfields.ColorField') },
						{ value: 'PasswordField', label: this.$root.trans.get('hierarchy::contentfields.PasswordField') },
						{ value: 'RelationField', label: this.$root.trans.get('hierarchy::contentfields.RelationField') },
						{ value: 'EmailField', label: this.$root.trans.get('hierarchy::contentfields.EmailField') }
					]
				}
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
			if(n.id) this.breadcrumbs.push({to: { name: 'contenttypes.fields', params: { id: n.id }}, text: n.name})
		}
	},
	methods: {
		requestStoreDynamic() {
			this.requestStore('contenttypes/' + this.resource.id + '/fields', 'contentfields.edit', this.resource.id)
		}
	}
};
</script>