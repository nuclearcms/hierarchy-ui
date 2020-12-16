<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">

		<ContentsTitle :resource="resource" :contentTitle="contentTitle" :contentTypeName="contentTypeName" :contentTypeRoute="contentTypeRoute"/>

		<div class="paper">
			<form method="POST" action="/api/contents" @submit.prevent="requestStore('contents/' + resource.id + '/transform', 'contents.edit', null, 'put')" autocomplete="off">
				
				<div class="paper__body paper__body--noside">
					<div class="paper__main">
						<FormBody :schema="schema" v-model="form" :readonly="false"/>
					</div>
				</div>

				<SubmitFooter :config="{icon: 'random'}" v-model="form"></SubmitFooter>

			</form>
		</div>
	</div>
</template>

<script>
import {Storer, Form, RequiresPermissions, assess_error} from 'umomega-foundation'
import ContentsTitle from '../../partials/ContentsTitle'
import ContentsEditHelper from '../../mixins/ContentsEditHelper'

export default {
	mixins: [ ContentsEditHelper, RequiresPermissions, Storer ],
	components: { ContentsTitle },
	data() { return {
		titleLabel: 'hierarchy::contents.transform_content',
		breadcrumbs: [
			{to: { name: 'contents.index'}, text: this.$root.trans.get('hierarchy::contents.multiple')}
		],
		guardedBy: 'write_contents',
		form: new Form({content_type_id: ''}),
		schema: [
			{
				type: 'SelectField',
				name: 'content_type_id',
				label: this.$root.trans.get('validation.attributes.type'),
				options: {
					required: true,
					choices: []
				}
			}
		]
	}},
	mounted() {
		const self = this

		// This is to override Storer's rendering
		setTimeout(function() {
			self.isLoaded = false
		}, 2)

		self.loadPretransform()
	},
	watch: {
		resource: function(resource) {
			var self = this

			this.breadcrumbs.push({to: { name: 'contents.edit', id: resource.id }, text: this.contentTitle })
		}
	},
	methods: {
		loadPretransform() {
			const self = this

			axios.get(api_url_with_token('contents/pretransform/' + self.$route.params.id))
				.then(function(response) {
					self.isLoaded = true

					if(response.data.action == 'populate')
					{
						const choices = response.data.types.map(function(t) {
							return { value: t.id, label: t.name }
						})

						self.schema[0].options.choices = choices

						if(choices.length == 1) self.form['content_type_id'] = choices[0].value
					} else if(response.data.action == 'redirect')
					{
						self.notifier.danger(response.data.message)
						router.push({ name: 'contents.edit', params: { id: self.$route.params.id } })
					}
				})
				.catch(function(error) { assess_error(error) })
		}
	}
};
</script>