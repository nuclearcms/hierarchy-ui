<template>
	<span>
		{{ trans.get('hierarchy::contentfields.event_destroyed') }}: <span class="has-color-danger">{{ activity.properties.name }}</span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>