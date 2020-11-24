<template>
	<Datable
		:defaultkey="'name'" :defaultdir="'asc'"
		:route="'contenttypes.index'"
		:sortables="[
			{key: 'name', label: 'validation.attributes.name'}
		]"
		:headers="['hierarchy::contentfields.multiple']"
		:createroutename="'contenttypes.create'"
		:indexloadroute="'contenttypes'"
		:searchloadroute="'contenttypes/search'"
		:bulkdeleteroute="'contenttypes/bulk'"
		:alllabel="'hierarchy::contenttypes.all'"
		:canwrite="$can('write_contenttypes')"
	>
		<template v-slot:default="slotProps">
			<td><router-link :to="{ name: 'contenttypes.edit', params: {id: slotProps.row.id} }" v-text="slotProps.row.name" /></td>
			<td><router-link :to="{ name: 'contenttypes.fields', params: {id: slotProps.row.id} }" v-text="trans.get('hierarchy::contentfields.multiple')" /></td>
			<DatableDropdown :can="$can('write_contenttypes')" :editroute="{ name: 'contenttypes.edit', params: {id: slotProps.row.id} }" :editlabel="trans.get('hierarchy::contenttypes.edit')" :deletepayload="{ bulk: false, route: 'contenttypes/' + slotProps.row.id }" :deletelabel="'hierarchy::contenttypes.delete'">
				<a href="#" class="dropdown-item" @click.prevent="duplicate('contenttypes/' + slotProps.row.id + '/duplicate')">
					<i class="icon fas fa-copy has-color-grey-darker"></i> {{ trans.get('hierarchy::contenttypes.duplicate') }}
				</a>
			</DatableDropdown>
		</template>
	</Datable>
</template>

<script>
import {View, Datable, DatableDropdown, RequiresPermissions, assess_error} from 'umomega-foundation'

export default {
	mixins: [ View, RequiresPermissions ],
	components: { Datable, DatableDropdown },
	data() { return {
		titleLabel: 'hierarchy::contenttypes.manage',
		guardedBy: 'read_contenttypes'
	}},
	methods: {
		duplicate(route) {
			const self = this

			axios.post(api_url_with_token(route))
				.then(function(response) {
					self.notifier.success(response.data.message)
					router.push({ name: 'contenttypes.edit', params: {id: response.data.payload.id} })
				})
				.catch(function(error) { assess_error(error) })
		}
	}
}
</script>