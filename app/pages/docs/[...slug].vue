<script setup lang="ts">
definePageMeta({
  layout: 'docs'
})

import { ref } from 'vue'
import type { ContentNavigationItem } from '@nuxt/content'

const route = useRoute()
const navigation = inject<Ref<ContentNavigationItem[]>>('navigation')

const showAI = ref(false)

const sortNavNaturally = (items: ContentNavigationItem[]): ContentNavigationItem[] => {
  return [...items].sort((a, b) => {
    const aVal = a.title || a.path || ''
    const bVal = b.title || b.path || ''
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

const { data: page } = await useAsyncData(route.path, () => queryCollection('docs').path(route.path).first())
if (!page.value) {
  throw createError({ statusCode: 404, statusMessage: 'Page not found', fatal: true })
}

const { data: surround } = await useAsyncData(`${route.path}-surround`, () => {
  return queryCollectionItemSurroundings('docs', route.path, {
    fields: ['description']
  })
})

const title = page.value.seo?.title || page.value.title
const description = page.value.seo?.description || page.value.description

useSeoMeta({
  title,
  ogTitle: title,
  description,
  ogDescription: description
})

defineOgImageComponent('Saas')
</script>

<template>
  <div v-if="page" class="w-full flex flex-col lg:flex-row px-4 sm:px-6 lg:px-8 max-w-full mx-auto relative">
    <!-- Left Sidebar -->
    <aside class="hidden lg:flex flex-col w-[250px] shrink-0 sticky top-[--header-height] h-[calc(100vh-var(--header-height))] overflow-y-auto py-8 pr-6 border-r border-white/5">
      <!-- Search -->
      <div class="mb-8">
        <UContentSearchButton
          :collapsed="false"
          label="Search..."
          class="w-full bg-white/5 hover:bg-white/10 text-zinc-400 border border-white/5 rounded-md"
        />
      </div>

      <!-- Navigation -->
      <div class="flex-1">
        <UContentNavigation
          :navigation="filteredNavigation"
          class="text-sm docus-nav"
        />
      </div>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 min-w-0 py-8 lg:px-12">
      <div class="max-w-3xl mx-auto">
        <!-- Breadcrumb / Category (Optional Docus style) -->
        <div class="mb-8 relative group">
          <p class="text-emerald-500 font-semibold text-sm mb-2 tracking-wide">{{ page.category || 'Getting Started' }}</p>
          <div class="flex items-center justify-between">
            <h1 class="text-4xl font-bold tracking-tight text-white mb-4">{{ page.title }}</h1>
            <UButton color="gray" variant="ghost" icon="i-lucide-copy" size="sm" class="opacity-0 group-hover:opacity-100 transition-opacity bg-white/5 border border-white/10 text-xs">Copy page</UButton>
          </div>
          <p v-if="page.description" class="text-lg text-zinc-400 font-light border-b border-white/5 pb-8">{{ page.description }}</p>
        </div>

        <!-- Markdown Content -->
        <div class="prose prose-invert prose-emerald prose-pre:bg-black/50 prose-pre:border prose-pre:border-white/5 max-w-none">
          <ContentRenderer
            v-if="page.body"
            :value="page"
          />
        </div>

        <USeparator v-if="surround?.length" class="my-12 border-white/5" />

        <UContentSurround :surround="surround" class="mt-8" />
      </div>
    </main>

    <!-- Right Sidebar -->
    <aside class="hidden xl:flex flex-col w-[250px] shrink-0 sticky top-[--header-height] h-[calc(100vh-var(--header-height))] overflow-y-auto py-8 pl-6">
      
      <!-- AI Chat Mode -->
      <DocsAiChat v-if="showAI" @close="showAI = false" />

      <!-- Default TOC & Community Mode -->
      <div v-else>
        <div v-if="page?.body?.toc?.links?.length" class="mb-10">
          <h3 class="text-sm font-semibold text-white mb-4">On this page</h3>
          <UContentToc :links="page.body.toc.links" class="docus-toc text-sm" />
        </div>
        
        <div>
          <h3 class="text-sm font-semibold text-white mb-4">Community</h3>
          <div class="flex flex-col gap-3 text-sm text-zinc-400">
            <NuxtLink to="https://ui.nuxt.com" target="_blank" class="flex items-center gap-2 hover:text-emerald-400 transition-colors">
              <UIcon name="i-lucide-book-open" class="w-4 h-4" />
              Nuxt UI docs <UIcon name="i-lucide-external-link" class="w-3 h-3 ml-auto opacity-50" />
            </NuxtLink>
            <NuxtLink to="https://content.nuxt.com" target="_blank" class="flex items-center gap-2 hover:text-emerald-400 transition-colors">
              <UIcon name="i-lucide-book-open" class="w-4 h-4" />
              Nuxt Content docs <UIcon name="i-lucide-external-link" class="w-3 h-3 ml-auto opacity-50" />
            </NuxtLink>
            <NuxtLink to="https://studio.nuxt.com" target="_blank" class="flex items-center gap-2 hover:text-emerald-400 transition-colors">
              <UIcon name="i-lucide-book-open" class="w-4 h-4" />
              Nuxt Studio docs <UIcon name="i-lucide-external-link" class="w-3 h-3 ml-auto opacity-50" />
            </NuxtLink>
            <div class="h-px w-full bg-white/5 my-2" />
            <button @click="showAI = true" class="flex items-center gap-2 hover:text-emerald-400 transition-colors text-left w-full">
              <UIcon name="i-lucide-bot" class="w-4 h-4" />
              Explain with AI
            </button>
          </div>
        </div>
      </div>

    </aside>
  </div>
</template>

<style scoped>
/* Docus-like Left Navigation Overrides */
:deep(.docus-nav a) {
  border-left: 1px solid transparent !important;
  border-radius: 0 !important;
  margin-left: 0 !important;
  padding-left: 1rem !important;
  color: #a1a1aa !important; /* zinc-400 */
}

:deep(.docus-nav a.active),
:deep(.docus-nav a.router-link-exact-active),
:deep(.docus-nav a[aria-current="page"]) {
  background-color: transparent !important;
  border-left-color: #10b981 !important; /* emerald-500 */
  color: #10b981 !important;
}

:deep(.docus-nav a:hover:not(.active):not(.router-link-exact-active)) {
  background-color: transparent !important;
  border-left-color: rgba(255,255,255,0.1) !important;
  color: #e4e4e7 !important; /* zinc-200 */
}

/* Hide the default pill background in Nuxt UI Pro UContentNavigation */
:deep(.docus-nav .bg-primary),
:deep(.docus-nav .bg-gray-100),
:deep(.docus-nav .bg-gray-800) {
  display: none !important;
}

/* Docus-like TOC Overrides */
:deep(.docus-toc ul) {
  border-left: 1px solid rgba(255,255,255,0.05);
  margin-left: 0.5rem;
}

:deep(.docus-toc a) {
  border-left: 1px solid transparent;
  margin-left: -1px;
  padding-left: 1rem !important;
  border-radius: 0 !important;
  color: #a1a1aa !important;
}

:deep(.docus-toc a.active),
:deep(.docus-toc a[class*="text-primary"]) {
  background-color: transparent !important;
  border-left-color: #10b981 !important;
  color: #10b981 !important;
}

:deep(.docus-toc a:hover:not(.active)) {
  border-left-color: rgba(255,255,255,0.2) !important;
  color: #e4e4e7 !important;
}
</style>
