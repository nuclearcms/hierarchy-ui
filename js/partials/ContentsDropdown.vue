<template>
	<div class="dropdown contents-dropdown is-hoverable is-right" v-if="can">
		<div class="dropdown-trigger">
			<slot name="trigger">
				<button class="button is-compact is-borderless" aria-haspopup="true">
					<i class="icon fas fa-ellipsis-v"></i>
				</button>
			</slot>
		</div>
		<div class="dropdown-menu" role="menu">
			<div class="dropdown-content">
				<div class="dropdown-flap" v-text="content.content_type.name" :style="'background-color: ' + content.content_type.color.hex"></div>
				<div class="dropdown-label" v-text="trans.get('foundation::general.options')"></div>
				<div v-if="!content.is_locked">
					<router-link v-if="!content.is_sterile && content.content_type.allowed_children_types.length > 0" class="dropdown-item" :to="{ name: 'contents.create', params: { parent: content.id }}">
						<i class="icon fas fa-plus has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.s_add_child') }}
					</router-link>
					<hr class="dropdown-divider" v-if="!content.is_sterile && content.content_type.allowed_children_types.length > 0">
					<router-link :to="{name: 'contents.edit', params: {id: content.id}}" class="dropdown-item">
						<i class="icon fas fa-edit has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.s_edit') }}
					</router-link>
					<a href="#" class="dropdown-item" @click.prevent="duplicate('contents/' + content.id + '/duplicate')">
						<i class="icon fas fa-copy has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.s_duplicate') }}
					</a>
					<a href="#" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal()">
						<i class="icon fas fa-trash"></i> {{ trans.get('hierarchy::contents.s_delete') }}
					</a>
					<hr class="dropdown-divider">
					<a href="#" class="dropdown-item" @click.prevent="toggleContentState('is_published', content.id)">
						<i :class="content.is_published ? 'icon fas fa-history has-color-grey-darker' : 'icon fas fa-check has-color-grey-darker'"></i> {{ trans.get(content.is_published ? 'hierarchy::contents.s_unpublish' : 'hierarchy::contents.s_publish') }}
					</a>
					<a href="#" class="dropdown-item" @click.prevent="toggleContentState('is_locked', content.id)">
						<i class="icon fas fa-lock has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.s_lock') }}
					</a>
				</div>
				<a v-else href="#" class="dropdown-item" @click.prevent="toggleContentState('is_locked', content.id)">
					<i class="icon fas fa-unlock has-color-grey-darker"></i> {{ trans.get('hierarchy::contents.s_unlock') }}
				</a>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	props: ['content', 'can'],
	methods: {
		toggleContentState(state, id) {
			const self = this

			axios.put(api_url_with_token('contents/' + id + '/state'), {state: state})
				.then(function(response) {
					self.notifier.success(response.data.message)
					Event.$emit('content-state-changed')
					
					if(response.data.event) Event.$emit(response.data.event, response.data)
				})
				.catch(function(error) { assess_error(error) })
		},
		openDeleteModal() {
			Event.$emit('delete-modal-open', { bulk: false, route: 'contents/' + this.content.id })
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
	}
}
</script>