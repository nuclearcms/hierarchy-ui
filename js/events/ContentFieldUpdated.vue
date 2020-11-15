<template>
	<span>
		{{ trans.get('hierarchy::contentfields.event_updated') }}<span v-if="activity.subject">: <router-link :to="{ name: 'contentfields.edit', params: {parent: activity.subject.content_type_id, id: activity.subject_id} }" v-text="activity.subject.label"></router-link></span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>