<script setup lang="ts">
import { computed, onMounted, ref, watchEffect } from 'vue';
import type { PortfolioData, Lang } from './types';
import type { FilterState, SortKey } from './filter';
import { filterAndSort } from './filter';
import { loadPortfolioData } from './data';
import { MESSAGES, detectDefaultLang } from './i18n';
import EntryCard from './EntryCard.vue';

// ---- state ----------------------------------------------------------

const data = ref<PortfolioData | null>(null);
const error = ref<string | null>(null);
const lang = ref<Lang>(readLangFromQuery() ?? detectDefaultLang());
const filter = ref<FilterState>(readQuery());

const m = computed(() => MESSAGES[lang.value]);

const visible = computed(() => {
  if (!data.value) return [];
  return filterAndSort(data.value.entries, filter.value, lang.value);
});

const stackMap = computed(() => {
  const m = new Map<string, { id: string; name: string; color: string }>();
  if (data.value) for (const s of data.value.stacks) m.set(s.id, s);
  return m;
});
const stageMap = computed(() => {
  const m = new Map<string, { id: string; icon: string; name: { ja: string; en: string } }>();
  if (data.value) for (const s of data.value.stages) m.set(s.id, s);
  return m;
});
const categoryMap = computed(() => {
  const m = new Map<string, { id: string; name: { ja: string; en: string } }>();
  if (data.value) for (const c of data.value.categories) m.set(c.id, c);
  return m;
});

// ---- lifecycle ------------------------------------------------------

onMounted(async () => {
  try {
    data.value = await loadPortfolioData();
  } catch (e) {
    error.value = e instanceof Error ? e.message : String(e);
  }
});

watchEffect(() => {
  writeQuery(filter.value, lang.value);
});

// ---- URL query sync -------------------------------------------------

function readLangFromQuery(): Lang | null {
  if (typeof window === 'undefined') return null;
  const q = new URLSearchParams(window.location.search);
  const v = q.get('lang');
  return v === 'ja' || v === 'en' ? v : null;
}

function readQuery(): FilterState {
  if (typeof window === 'undefined') return defaultFilter();
  const q = new URLSearchParams(window.location.search);
  return {
    query: q.get('q') ?? '',
    category: q.get('category') ?? 'all',
    stack: q.get('stack') ?? 'all',
    stage: q.get('stage') ?? 'all',
    sort: (q.get('sort') as SortKey) ?? 'number',
  };
}

function writeQuery(f: FilterState, l: Lang) {
  if (typeof window === 'undefined') return;
  const q = new URLSearchParams();
  if (f.query) q.set('q', f.query);
  if (f.category !== 'all') q.set('category', f.category);
  if (f.stack !== 'all') q.set('stack', f.stack);
  if (f.stage !== 'all') q.set('stage', f.stage);
  if (f.sort !== 'number') q.set('sort', f.sort);
  q.set('lang', l);
  const qs = q.toString();
  const url = qs ? `${window.location.pathname}?${qs}` : window.location.pathname;
  window.history.replaceState(null, '', url);
}

function defaultFilter(): FilterState {
  return { query: '', category: 'all', stack: 'all', stage: 'all', sort: 'number' };
}
</script>

<template>
  <div v-if="error" class="state state-error">
    {{ m.errorLabel }}: {{ error }}
  </div>

  <div v-else-if="!data" class="state state-loading">
    {{ m.loadingLabel }}
  </div>

  <template v-else>
    <header class="site-header">
      <div class="header-top">
        <a class="home-link" href="/">{{ m.homeLabel }}</a>
        <div class="lang-switch">
          <label>{{ m.langLabel }}</label>
          <select v-model="lang">
            <option value="ja">JA</option>
            <option value="en">EN</option>
          </select>
        </div>
      </div>
      <h1>{{ m.title }}</h1>
      <p class="subtitle">{{ m.subtitle }}</p>
      <p class="meta">
        {{ m.totalCount(data.entries.length) }}　·　{{ m.lastUpdated(data.updatedAt) }}　·　{{ m.framework }}
      </p>
    </header>

    <main>
      <section class="controls">
        <input
          type="text"
          class="search"
          :placeholder="m.searchPlaceholder"
          v-model="filter.query"
        />

        <div class="filters">
          <label class="select-wrap">
            <span class="select-label">{{ m.categoryLabel }}</span>
            <select v-model="filter.category">
              <option value="all">{{ m.allLabel }}</option>
              <option v-for="c in data.categories" :key="c.id" :value="c.id">{{ c.name[lang] }}</option>
            </select>
          </label>
          <label class="select-wrap">
            <span class="select-label">{{ m.stackLabel }}</span>
            <select v-model="filter.stack">
              <option value="all">{{ m.allLabel }}</option>
              <option v-for="s in data.stacks" :key="s.id" :value="s.id">{{ s.name }}</option>
            </select>
          </label>
          <label class="select-wrap">
            <span class="select-label">{{ m.stageLabel }}</span>
            <select v-model="filter.stage">
              <option value="all">{{ m.allLabel }}</option>
              <option v-for="s in data.stages" :key="s.id" :value="s.id">
                {{ s.icon }} {{ s.name[lang] }}
              </option>
            </select>
          </label>
          <label class="select-wrap">
            <span class="select-label">{{ m.sortLabel }}</span>
            <select v-model="filter.sort">
              <option value="number">{{ m.sortNumber }}</option>
              <option value="newest">{{ m.sortNewest }}</option>
              <option value="oldest">{{ m.sortOldest }}</option>
              <option value="name">{{ m.sortName }}</option>
            </select>
          </label>
        </div>

        <div class="result-count">
          {{ m.filteredCount(visible.length, data.entries.length) }}
        </div>
      </section>

      <p v-if="visible.length === 0" class="empty">{{ m.noResults }}</p>

      <section v-else class="grid">
        <EntryCard
          v-for="entry in visible"
          :key="entry.slug"
          :entry="entry"
          :lang="lang"
          :stack-map="stackMap"
          :stage-map="stageMap"
          :category-map="categoryMap"
          :messages="m"
        />
      </section>
    </main>

    <footer class="site-footer">
      <span>SEN 合同会社 · Vue 3 (Composition API + Vite + TypeScript) · entry 022</span>
    </footer>
  </template>
</template>

<style scoped>
/* Component-scoped styles are empty — global styles live in style.css */
</style>
