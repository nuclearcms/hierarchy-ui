<template>
	<PaperHeader :permission="permission" :resource="resource" :editingLocale="editingLocale" :errors="errors">
		<div v-if="!resource.is_locked">
			<router-link v-if="canHaveMoreTranslations" class="dropdown-item" :to="{ name: 'contents.translate', params: {id: this.contentId} }">
				<i class="icon fas fa-language has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.translate') }}
			</router-link>
			<a href="#" v-if="canDeleteCurrentTranslation" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal({ bulk: false, route: 'contents/' + contentId + '/translate/' + editingLocale})">
				<i class="icon fas fa-minus"></i> {{ trans.get('foundation::general.delete_translation') }}
			</a>
			<a href="#" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal({ bulk: false, route: 'contents/' + contentId})">
				<i class="icon fas fa-trash"></i> {{ trans.get('hierarchy::contents.delete') }}
			</a>
			<hr class="dropdown-divider" v-if="!resource.is_sterile && resource.content_type && resource.content_type.allowed_children_types.length > 0">
			<router-link v-if="!resource.is_sterile && resource.content_type && resource.content_type.allowed_children_types.length > 0" class="dropdown-item" :to="{ name: 'contents.create', params: { parent: contentId }}">
				<i class="icon fas fa-plus has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.add_child') }}
			</router-link>
			<hr class="dropdown-divider">
			<a href="#" class="dropdown-item" @click.prevent="duplicate('contents/' + contentId + '/duplicate')">
				<i class="icon fas fa-copy has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.duplicate') }}
			</a>
			<router-link v-if="canTransform" class="dropdown-item" :to="{ name: 'contents.transform', params: { id: contentId }}">
				<i class="icon fas fa-random has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.transform_content') }}
			</router-link>
			<hr class="dropdown-divider">
			<a href="#" class="dropdown-item" @click.prevent="toggleContentState('is_published')">
				<i :class="resource.is_published ? 'icon fas fa-history has-color-grey-darker' : 'icon fas fa-check has-color-grey-darker'"></i> {{ trans.get(resource.is_published ? 'hierarchy::contents.unpublish_content' : 'hierarchy::contents.publish_content') }}
			</a>
			<a href="#" class="dropdown-item" @click.prevent="toggleContentState('is_locked')">
				<i class="icon fas fa-lock has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.lock_content') }}
			</a>
		</div>
		<a v-else href="#" class="dropdown-item" @click.prevent="toggleContentState('is_locked')">
			<i class="icon fas fa-unlock has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.unlock_content') }}
		</a>
	</PaperHeader>
</template>

<script>
import {PaperHeader, assess_error} from 'umomega-foundation'

export default {
	components: { PaperHeader },
	props: ['permission', 'resource', 'editingLocale', 'errors', 'canHaveMoreTranslations', 'canDeleteCurrentTranslation', 'contentId'],
	methods: {
		toggleContentState(state) {
			const self = this

			axios.put(api_url_with_token('contents/' + self.$route.params.id + '/state'), {state: state})
				.then(function(response) {
					self.notifier.success(response.data.message)
					Event.$emit('content-state-changed')

					if(response.data.event) Event.$emit(response.data.event, response.data)
				})
				.catch(function(error) { assess_error(error) })
		},
		openDeleteModal(payload) {
			Event.$emit('delete-modal-open', payload)
		},
		duplicate(route) {
			const self = this

			axios.post(api_url_with_token(route))
				.then(function(response) {
					self.notifier.success(response.data.message)
					router.push({ name: 'contents.edit', params: {id: response.data.payload.id} })
					Event.$emit('content-tree-modified')
				})
				.catch(function(error) { assess_error(error) })
		}
	},
	computed: {
		canTransform() {
			if(this.resource.ancestors != undefined && this.resource.ancestors.length > 0) {
				const parentContentType = this.resource.ancestors[this.resource.ancestors.length - 1].content_type

				if((parentContentType.allowed_children_types.includes(this.resource.content_type_id) && parentContentType.allowed_children_types.length > 1) || (!parentContentType.allowed_children_types.includes(this.resource.content_type_id) && parentContentType.allowed_children_types.length > 0)) return true

				return false
			}
			
			return true
		}
	}
}
</script>