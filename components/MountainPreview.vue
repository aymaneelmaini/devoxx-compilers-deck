<script setup lang="ts">
// Shows the compiler mountain with a red circle around the stage about to be covered.
// Coordinates are in the PNG's pixel space (1824 x 833), so they scale with the image.
const VIEW_W = 1824
const VIEW_H = 833

type Circle = { cx: number; cy: number; r: number }

const targets: Record<string, Circle[]> = {
  scanning: [{ cx: 300, cy: 600, r: 140 }],
  parsing: [
    { cx: 385, cy: 320, r: 110 },
    { cx: 652, cy: 290, r: 55 },
  ],
  analysis: [{ cx: 660, cy: 95, r: 125 }],
  optimizing: [{ cx: 870, cy: 270, r: 125 }],
  codegen: [
    { cx: 1340, cy: 310, r: 160 },
    { cx: 1066, cy: 258, r: 55 },
  ],
  runtime: [{ cx: 1400, cy: 700, r: 120 }],
}

const props = defineProps<{ focus: string; caption?: string }>()
</script>

<template>
  <div class="mountain-preview flex flex-col items-center gap-6">
    <div class="relative inline-block">
      <img
        src="/crafting-interpreters-pipeline.png"
        alt="Compiler pipeline as a mountain"
        class="block max-h-[60vh] w-auto"
      />
      <svg
        :viewBox="`0 0 ${VIEW_W} ${VIEW_H}`"
        class="pointer-events-none absolute inset-0 h-full w-full"
      >
        <circle
          v-for="(c, i) in targets[props.focus] ?? []"
          :key="i"
          :cx="c.cx"
          :cy="c.cy"
          :r="c.r"
          fill="none"
          stroke="#e11d48"
          stroke-width="10"
          class="highlight"
        />
      </svg>
    </div>
    <div v-if="props.caption" class="text-xl opacity-80">{{ props.caption }}</div>
  </div>
</template>

<style scoped>
.highlight {
  animation: pulse 1.6s ease-in-out infinite;
  transform-box: fill-box;
  transform-origin: center;
}
@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.55; transform: scale(1.06); }
}
</style>
