<script setup lang="ts">
import type { mastodon } from 'masto'

const { tag } = defineProps<{
  tag: mastodon.v1.Tag
}>()

const emit = defineEmits<{
  (event: 'change'): void
}>()

const { client } = $(useMasto())

const toggleFollowTag = async () => {
  // We save the state so be can do an optimistic UI update, but fallback to the previous state if the API call fails
  const previousFollowingState = tag.following

  // eslint-disable-next-line vue/no-mutating-props
  tag.following = !tag.following

  try {
    if (previousFollowingState)
      await client.v1.tags.unfollow(tag.name)
    else
      await client.v1.tags.follow(tag.name)

    emit('change')
  }
  catch (error) {
    // eslint-disable-next-line vue/no-mutating-props
    tag.following = previousFollowingState
  }
}
</script>

<template>
  <button
    rounded group focus:outline-none
    hover:text-primary focus-visible:text-primary
    :aria-label="tag.following ? $t('tag.unfollow_label', [tag.name]) : $t('tag.follow_label', [tag.name])"
    @click="toggleFollowTag()"
  >
    <CommonTooltip placement="bottom" :content="tag.following ? $t('tag.unfollow') : $t('tag.follow')">
      <div rounded-full p2 elk-group-hover="bg-orange/10" group-focus-visible="bg-orange/10" group-focus-visible:ring="2 current">
        <div :class="[tag.following ? 'i-ri:star-fill' : 'i-ri:star-line']" />
      </div>
    </CommonTooltip>
  </button>
</template>
