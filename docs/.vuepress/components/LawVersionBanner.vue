<script setup>
import { computed } from "vue";

const props = defineProps({
  data: {
    type: Object,
    required: true,
  },
});

const selected = computed(() =>
  props.data.versions.find((version) => version.id === props.data.selectedVersionId)
);

const showBanner = computed(() => {
  if (!selected.value) return false;
  return selected.value.status !== "current" || Boolean(selected.value.effectiveUntil);
});
</script>

<template>
  <section
    v-if="showBanner"
    class="law-version-banner"
    :class="[
      `is-${selected.status}`,
      { 'is-ending': selected.status === 'current' && selected.effectiveUntil },
    ]"
    role="note"
    :aria-label="
      selected.status === 'pending'
        ? '尚未生效版本'
        : selected.status === 'expired'
          ? '历史版本'
          : '即将失效版本'
    "
  >
    <strong>
      {{
        selected.status === "pending"
          ? "尚未生效"
          : selected.status === "expired"
            ? "历史版本"
            : "即将废止"
      }}
    </strong>
    <span v-if="selected.status === 'pending'">
      本版本于<time :datetime="selected.promulgatedOn">{{ selected.promulgatedOn }}</time
      >公布，将于<time :datetime="selected.effectiveFrom">{{ selected.effectiveFrom }}</time
      >起施行。
    </span>
    <span v-else-if="selected.status === 'expired'">
      本版本有效期至<time :datetime="selected.effectiveUntil">{{ selected.effectiveUntil }}</time
      >。
    </span>
    <span v-else>
      本版本有效期至<time :datetime="selected.effectiveUntil">{{ selected.effectiveUntil }}</time
      >；自次日起
      <template v-if="data.repealedBy">
        请参见<RouterLink :to="data.repealedBy.path">{{ data.repealedBy.title }}</RouterLink>。
      </template>
      <template v-else>将不再作为现行有效版本。</template>
    </span>
  </section>
</template>
