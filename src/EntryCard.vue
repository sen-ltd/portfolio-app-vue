<script setup lang="ts">
import { computed } from 'vue';
import type { Entry, Lang, LocalizedText } from './types';
import { GitHubIcon, XIcon, QiitaIcon, DevToIcon } from './icons';

const props = defineProps<{
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

// Resolve icon-row link targets once per render; mirrors IconLinks in the
// React reference. `sen` entries produce the JA article badge only when no
// qiita article exists (avoids duplicating the same JA destination).
const qiitaArticle = computed(() =>
  props.entry.articles.find((a) => a.platform === 'qiita'),
);
const devtoArticle = computed(() =>
  props.entry.articles.find((a) => a.platform === 'devto'),
);
const senArticle = computed(() =>
  props.entry.articles.find((a) => a.platform === 'sen'),
);
const twitterUrl = computed(() => props.entry.social?.twitter);

const showIconLinks = computed(
  () =>
    Boolean(props.entry.github) ||
    Boolean(twitterUrl.value) ||
    Boolean(qiitaArticle.value) ||
    Boolean(devtoArticle.value) ||
    Boolean(senArticle.value),
);

const showJaBadge = computed(
  () => Boolean(senArticle.value) && !qiitaArticle.value,
);

const jaBadgeLabel = computed(() => (props.lang === 'ja' ? '記事' : 'JA'));
const jaBadgeTitle = computed(() =>
  props.lang === 'ja' ? 'JA 記事' : 'Japanese article',
);
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

      <div v-if="showIconLinks" class="icon-links">
        <a
          v-if="entry.github"
          :href="entry.github"
          class="icon-link"
          target="_blank"
          rel="noopener"
          title="GitHub"
        >
          <GitHubIcon />
        </a>
        <a
          v-if="twitterUrl"
          :href="twitterUrl"
          class="icon-link"
          target="_blank"
          rel="noopener"
          title="X (Twitter)"
        >
          <XIcon />
        </a>
        <a
          v-if="qiitaArticle"
          :href="qiitaArticle.url"
          class="icon-link"
          target="_blank"
          rel="noopener"
          title="Qiita"
        >
          <QiitaIcon />
        </a>
        <a
          v-if="devtoArticle"
          :href="devtoArticle.url"
          class="icon-link"
          target="_blank"
          rel="noopener"
          title="dev.to"
        >
          <DevToIcon />
        </a>
        <a
          v-if="showJaBadge && senArticle"
          :href="senArticle.url"
          class="article-badge"
          target="_blank"
          rel="noopener"
          :title="jaBadgeTitle"
        >
          {{ jaBadgeLabel }}
        </a>
      </div>

      <span v-if="entry.testCount && entry.testCount > 0" class="tests-badge">
        {{ messages.testsLabel(entry.testCount) }}
      </span>
    </div>
  </article>
</template>
