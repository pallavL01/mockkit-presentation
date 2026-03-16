<script setup>
import { slides } from '#slidev/slides'
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { go } = useNav()

const tocItems = computed(() =>
  slides.value
    .filter(r =>
      r.meta?.slide?.title &&
      !r.meta?.slide?.frontmatter?.hideInToc
    )
    .map((r, i) => ({
      num: String(i + 1),
      label: r.meta.slide.title,
      no: r.no,
      color: r.meta.slide.frontmatter?.tocColor ?? null,
      extra: r.meta.slide.frontmatter?.tocGroup === 'extra',
    }))
)

const mainItems = computed(() => tocItems.value.filter(i => !i.extra))
const extraItems = computed(() => tocItems.value.filter(i => i.extra))
</script>

<template>
  <div class="toc-grid">
    <div
      v-for="item in mainItems"
      :key="item.no"
      class="toc-item"
      @click="go(item.no)"
    >
      <div class="toc-num" :style="item.color ? { background: item.color } : {}">
        {{ item.num }}
      </div>
      <span class="toc-label">{{ item.label }}</span>
    </div>
  </div>
  <div v-if="extraItems.length" class="toc-extras">
    <div
      v-for="item in extraItems"
      :key="item.no"
      class="toc-item"
      @click="go(item.no)"
    >
      <div class="toc-num" :style="item.color ? { background: item.color } : {}">
        {{ item.num }}
      </div>
      <span class="toc-label">{{ item.label }}</span>
    </div>
  </div>
</template>

<style scoped>
.toc-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 8px;
  margin-top: 10px;
}
.toc-extras {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 8px;
}
.toc-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  background: var(--card);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.15s, box-shadow 0.15s;
}
.toc-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}
.toc-num {
  width: 28px;
  height: 28px;
  background: var(--slate-700);
  color: #e2e8f0;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 700;
  flex-shrink: 0;
}
.toc-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--slate-800);
}
</style>
