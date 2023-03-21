<script setup lang="ts">
// @todo Move to separate store UI layer later once @see https://github.com/nuxt/nuxt/issues/13367 is resolved
import useSplitTesting from 'nuxt-store-core-app/src/composables/useSplitTesting'
import { useBuilderStore } from 'nuxt-store-core-app/src/store/builder'
import { Widget } from 'nuxt-store-core-app/src/types/widget'
import WidgetComponent from './WidgetComponent.vue'

const props = defineProps<{
  slot: string
  version?: 'A' | 'B'
  params?: Record<string, string>
}>()

const splitTestingOptions = {
  version: props.version,
  params: props.params,
  page: props.slot.replace('page_', ''),
}
const splitTesting =
  props.slot.startsWith('page_') && props.version
    ? useSplitTesting(splitTestingOptions)
    : undefined

const builderStore = useBuilderStore()
builderStore.setSlot(props.slot)
if (props.params) {
  builderStore.setParams(props.params)
}

const widgets = await builderStore.fetchWidgets(splitTesting)

const calculateWidgetSize = (widget: Widget): string => {
  return `col-span-${widget.cols} row-span-${widget.rows}`
}

const calculateWidgetPosition = (widget: Widget): string => {
  if (widget.colStart === undefined) {
    return ''
  }
  return `col-start-${widget.colStart} row-start-${widget.rowStart}`
}

const widgetList = computed(() => {
  return widgets.map((widget: any) => ({
    ...widget,
    position: calculateWidgetPosition(widget.attributes),
    size: calculateWidgetSize(widget.attributes),
  }))
})
</script>

<template>
  <div class="grid grid-cols-1 md:grid-cols-12 md:gap-y-10 md:gap-x-4">
    <template v-for="widget in widgetList" :key="widget.id">
      <div :class="[widget.position, widget.size]">
        <WidgetComponent :widget="widget.attributes">
          <template #beforeComponent>
            <slot :widget="widget.attributes" name="beforeComponent" />
          </template>
          <!-- Main slot content if any here -->
          <template #afterComponent>
            <slot :widget="widget.attributes" name="afterComponent" />
          </template>
        </WidgetComponent>
      </div>
    </template>
  </div>
</template>
