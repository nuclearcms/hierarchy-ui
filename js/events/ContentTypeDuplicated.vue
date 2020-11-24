<template>
	<span>
		{{ trans.get('hierarchy::contenttypes.event_duplicated') }}<span v-if="activity.subject">: <router-link :to="{ name: 'contenttypes.edit', params: {id: activity.subject_id} }" v-text="activity.subject.name"></router-link></span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>