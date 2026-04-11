<script setup lang="ts">
import type { Entry, Lang, LocalizedText } from './types';

defineProps<{
  entry: Entry;
  lang: Lang;
  stackMap: Map<string, { id: string; name: string; color: string }>;
  stageMap: Map<string, { id: string; icon: string; name: LocalizedText }>;
  categoryMap: Map<string, { id: string; name: LocalizedText }>;
  messages: {
    demoLabel: string;
    githubLabel: string;
    testsLabel: (n: number) => string;
  };
}>();
</script>

<template>
  <article class="card">
    <div class="card-head">
      <span class="entry-number">#{{ String(entry.number).padStart(3, '0') }}</span>
      <span v-if="stageMap.get(entry.stage)" class="stage-badge">
        {{ stageMap.get(entry.stage)?.icon }}
        {{ stageMap.get(entry.stage)?.name[lang] }}
      </span>
    </div>

    <h2 class="entry-name">{{ entry.name[lang] }}</h2>

    <div v-if="categoryMap.get(entry.category)" class="category">
      {{ categoryMap.get(entry.category)?.name[lang] }}
    </div>

    <p class="pitch">{{ entry.pitch[lang] }}</p>

    <div class="tech-row">
      <span
        v-for="techId in entry.tech"
        :key="techId"
        class="tech-chip"
        :style="{ borderLeftColor: stackMap.get(techId)?.color ?? '#272b35' }"
      >
        {{ stackMap.get(techId)?.name ?? techId }}
      </span>
    </div>

    <div class="actions">
      <a
        v-if="entry.demo"
        :href="entry.demo"
        class="action-btn primary"
        target="_blank"
        rel="noopener"
      >
        ↗ {{ messages.demoLabel }}
      </a>
      <a
        v-if="entry.github"
        :href="entry.github"
        class="action-btn"
        target="_blank"
        rel="noopener"
      >
        {{ messages.githubLabel }}
      </a>
      <div v-if="entry.articles.length > 0" class="articles">
        <a
          v-for="a in entry.articles"
          :key="a.url"
          :href="a.url"
          class="article-link"
          target="_blank"
          rel="noopener"
        >
          {{ a.platform }}
        </a>
      </div>
      <span v-if="entry.testCount && entry.testCount > 0" class="tests-badge">
        {{ messages.testsLabel(entry.testCount) }}
      </span>
    </div>
  </article>
</template>
