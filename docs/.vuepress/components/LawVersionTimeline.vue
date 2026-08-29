<script setup>
import { computed } from "vue";

const props = defineProps({
  data: {
    type: Object,
    required: true,
  },
  compact: {
    type: Boolean,
    default: false,
  },
});

const selected = computed(() =>
  props.data.versions.find((version) => version.id === props.data.selectedVersionId)
);

const statusLabel = (status) => {
  if (status === "expired" && props.data.repealedBy) return "已废止";
  return ({ current: "现行有效", pending: "尚未生效", expired: "历史版本" })[status];
};

const shortLawTitle = (title) => title.replace(/^中华人民共和国/, "");

const versionStateClasses = (version) => [
  `is-${version.status}`,
  {
    "is-selected": version.id === props.data.selectedVersionId,
    "is-repealed": version.status === "expired" && Boolean(props.data.repealedBy),
  },
];

const periodLabel = (version) => {
  if (version.status === "pending") return `${version.effectiveFrom} 起施行`;
  if (version.effectiveUntil) {
    return `${version.effectiveFrom} — ${version.effectiveUntil}`;
  }
  return `${version.effectiveFrom} 起施行`;
};
</script>

<template>
  <div v-if="compact" class="law-version-compact">
    <details class="law-version-selector" :class="selected ? versionStateClasses(selected) : []">
      <summary>
        <span>法律版本</span>
        <strong v-if="selected">
          {{ selected.label }} ·
          <span class="law-version-summary-status">{{ statusLabel(selected.status) }}</span>
        </strong>
      </summary>
      <ol class="law-version-list">
        <li
          v-for="version in data.versions"
          :key="version.id"
          class="law-version-item"
          :class="versionStateClasses(version)"
        >
          <RouterLink
            :to="version.path"
            class="law-version-link"
            :aria-current="version.id === data.selectedVersionId ? 'page' : undefined"
          >
            <span class="law-version-dot" aria-hidden="true"></span>
            <span class="law-version-copy">
              <span class="law-version-status">{{ statusLabel(version.status) }}</span>
              <strong>{{ version.label }}</strong>
              <time>{{ periodLabel(version) }}</time>
            </span>
          </RouterLink>
        </li>
      </ol>
    </details>

    <details v-if="data.repeals.length" class="law-lineage-selector">
      <summary>
        <span>立法沿革</span>
        <strong>同时废止的法律（{{ data.repeals.length }}）</strong>
      </summary>
      <div class="law-lineage-content">
        <p>本法施行时同时废止：</p>
        <ul class="law-lineage-list">
          <li v-for="law in data.repeals" :key="law.path">
            <RouterLink :to="law.path" class="law-lineage-link">{{ shortLawTitle(law.title) }}</RouterLink>
          </li>
        </ul>
      </div>
    </details>
  </div>

  <div v-else class="law-version-timeline">
    <section class="law-version-section">
      <h2>法律版本</h2>
      <ol class="law-version-list">
        <li
          v-for="version in data.versions"
          :key="version.id"
          class="law-version-item"
          :class="versionStateClasses(version)"
        >
          <RouterLink
            :to="version.path"
            class="law-version-link"
            :aria-current="version.id === data.selectedVersionId ? 'page' : undefined"
          >
            <span class="law-version-dot" aria-hidden="true"></span>
            <span class="law-version-copy">
              <span class="law-version-status">{{ statusLabel(version.status) }}</span>
              <strong>{{ version.label }}</strong>
              <time>{{ periodLabel(version) }}</time>
            </span>
          </RouterLink>
        </li>
      </ol>
    </section>

    <section v-if="data.repeals.length" class="law-lineage-section">
      <h2>立法沿革</h2>
      <p>本法施行时同时废止：</p>
      <ul class="law-lineage-list">
        <li v-for="law in data.repeals" :key="law.path">
          <RouterLink :to="law.path" class="law-lineage-link">{{ shortLawTitle(law.title) }}</RouterLink>
        </li>
      </ul>
    </section>
  </div>
</template>
