<template>
	<span>
		{{ trans.get('hierarchy::contents.event_destroyed_translation') }}<span v-if="activity.subject">: <router-link :to="{ name: 'contents.edit', params: {id: activity.subject_id} }" v-text="activity.subject.title[$root.appLocale]"></router-link></span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>