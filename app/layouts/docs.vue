<script setup lang="ts">
import type { ContentNavigationItem } from '@nuxt/content'

const route = useRoute()
const navigation = inject<Ref<ContentNavigationItem[]>>('navigation')

const sortNavNaturally = (items: ContentNavigationItem[]): ContentNavigationItem[] => {
  return [...items].sort((a, b) => {
    const aVal = a.path || a.title || ''
    const bVal = b.path || b.title || ''
    return aVal.localeCompare(bVal, undefined, { numeric: true, sensitivity: 'base' })
  }).map(item => {
    if (item.children) {
      return { ...item, children: sortNavNaturally(item.children) }
    }
    return item
  })
}

const filteredNavigation = computed(() => {
  const currentFramework = route.path.split('/')[2] // 'nuxt' or 'laravel'
  if (!currentFramework || !navigation?.value) return []

  const children = navigation.value.find(item => item.path === `/docs/${currentFramework}`)?.children || []
  return sortNavNaturally(children)
})
</script>

<template>
  <div>
    <AppHeader />
    <DocsHeader />

    <UMain>
      <slot />
    </UMain>

    <AppFooter />
  </div>
</template>
