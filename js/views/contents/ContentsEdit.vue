<template>
	<div>
		<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
			
			<ContentsTitle :resource="resource" :contentTitle="contentTitle" :contentTypeName="contentTypeName" :contentTypeRoute="contentTypeRoute"/>

			<tabs class="is-marginless" :tabs="tabs"></tabs>

			<div class="paper">
				<ContentsHeader :permission="$can('write_contents')" :resource="resource" :editingLocale="editingLocale" :errors="form.errors" :canHaveMoreTranslations="canHaveMoreTranslations" :canDeleteCurrentTranslation="canDeleteCurrentTranslation" :contentId="contentId"/>

				<form method="POST" action="/api/contents" @submit.prevent="requestUpdate('contents')" @change="clearError($event.target.name)" autocomplete="off">
					
					<div class="paper__body">
						<div class="paper__main">
							<div v-for="locale in resource.locales">
								<FormBody :class="locale == editingLocale ? '' : 'is-sr-only'" :schema="schema" v-model="form" :readonly="!$can('write_contents') || resource.is_locked == 1" :translatable="true" :translatableFields="translatableFields" :locale="locale"/>
							</div>
						</div>
						<div class="paper__side">

							<div class="mb-xl" v-if="contentTaggable">
								<h3 class="is-size-5 has-color-grey mb-sm" v-text="trans.get('tags::tags.multiple')"></h3>

								<ContentsTags :errors="form.errors" :readonly="!$can('write_contents') || resource.is_locked == 1" :value="form['tags']"/>
							</div>

							<h3 class="is-size-5 has-color-grey mb-sm" v-text="trans.get('hierarchy::contents.seo_and_search')"></h3>
							<div v-for="locale in resource.locales">
								<FormBody :class="locale == editingLocale ? '' : 'is-sr-only'" :schema="schemaSecondary" v-model="form" :readonly="!$can('write_contents') || resource.is_locked == 1" :translatable="true" :translatableFields="secondaryTranslatableFields" :locale="locale"/>
							</div>

						</div>
					</div>

					<SubmitFooter v-if="$can('write_contents') && resource.is_locked == 0" :config="{icon: 'save'}" v-model="form">
						<div class="control" v-if="contentStatus < 50">
							<button class="button icon-only is-warning" type="button" @click.prevent="saveAndPublish" :disabled="form.errors.anyErrors">
								<i class="icon fas fa-check"></i>
							</button>
						</div>
					</SubmitFooter>
					<div class="paper__footer" v-else>
						<div class="field has-addons is-pulled-right">
							<div class="control">
								<button class="button icon-only-wide is-dark" type="submit">
									<i class="icon fas fa-lock"></i>
								</button>
							</div>
						</div>
					</div>
					
				</form>
			</div>
		</div>
		<FloatingLibrary />
	</div>
</template>

<script>
import {Updater, Tabs, Form, RequiresPermissions, Translatable, FloatingLibrary} from 'umomega-foundation'
import ContentsTitle from '../../partials/ContentsTitle'
import ContentsHeader from '../../partials/ContentsHeader'
import ContentsTags from '../../partials/ContentsTags'
import ContentsEditHelper from '../../mixins/ContentsEditHelper'

export default {
	mixins: [ Updater, RequiresPermissions, Translatable, ContentsEditHelper ],
	components: { Tabs, ContentsHeader, ContentsTitle, ContentsTags, FloatingLibrary },
	data() { return {
		titleLabel: 'hierarchy::contents.edit',
		hasOptions: true,
		breadcrumbs: [
			{to: { name: 'contents.index'}, text: this.$root.trans.get('hierarchy::contents.multiple')}
		],
		guardedBy: 'read_contents',
		tabs: [
			{route: 'contents.edit', label: 'hierarchy::contents.single', active: true},
			{route: 'contents.settings', label: 'hierarchy::contents.settings', active: false},
			{route: 'contents.statistics', label: 'hierarchy::contents.statistics', active: false},
		],
		translatableFields: ['title'],
		secondaryTranslatableFields: ['meta_title', 'meta_description', 'meta_author', 'keywords', 'cover_image'],
		form: new Form({content_type_id: '', title: {}, cover_image: {}, meta_title: {}, meta_description: {}, meta_author: {}, keywords: {}, status: 100, tags: []}),
		preventPopulateForm: true,
		schema: [
			{
				type: 'TextField',
				name: 'title',
				label: this.$root.trans.get('validation.attributes.title'),
				options: {required: true}
			}
		],
		schemaSecondary: [
			{
				type: 'MediaField',
				name: 'cover_image',
				label: this.$root.trans.get('validation.attributes.cover_image'),
				options: {multiple: false, filters: ['image']}
			},
			{
				type: 'TextField',
				name: 'meta_title',
				label: this.$root.trans.get('validation.attributes.meta_title')
			},
			{
				type: 'TextareaField',
				name: 'meta_description',
				label: this.$root.trans.get('validation.attributes.meta_description')
			},
			{
				type: 'TextareaField',
				name: 'keywords',
				label: this.$root.trans.get('validation.attributes.keywords')
			},
			{
				type: 'TextField',
				name: 'meta_author',
				label: this.$root.trans.get('validation.attributes.meta_author')
			}
		]
	}},
	watch: {
		$route(to, from) {
			if(from.params.id != to.params.id) {
				this.loadResource()
				window.scrollTo(0, 0)
			}
			if(from.params.locale != to.params.locale) this.editingLocale = to.params.locale
		}
	},
	methods: {
		saveAndPublish() {
			this.form.status = 50
			this.requestUpdate('contents')
		},
		registerResourceLoaded(self) {
			Event.$off('resource-loaded')
			Event.$on('resource-loaded', function(data) {
				self.schema.length = 1
				self.translatableFields.length = 1
				self.editingLocale = self.$root.appLocale

				data.schema.schema.forEach(function(field, i) {
					self.translatableFields.push(field.name)
					self.schema.push(field)
				})

				// We do this so that we don't send the data from any other previous form
				self.form.originalData = data
				self.form.populate(data)
				
				if((data.content_type.hides_children || data.hides_children) && self.tabls.length == 3) {
					self.tabs.unshift({ route: 'contents.children', label: 'hierarchy::contents.children', active: false})
				} else if(self.tabs.length > 3) {
					self.tabs.shift()
				}
			})
		}
	},
	mounted() {
		const self = this

		Event.$on('tag-field-changed', function(data) {
			self.$set(self.form, 'tags', data)

			Event.$emit('input', self.form)
		})

		self.registerResourceLoaded(self)
	},
	beforeDestroy() {
		Event.$off('resource-loaded')
		Event.$off('tag-field-changed')
	}
}
</script>