<script setup lang="ts">
const config = useRuntimeConfig()
const props = defineProps<{
  widget: Record<string, any>
}>()

console.log(config.public.basePath, 'basePath')

// @@ts-ignore
// @todo need to check if file exists otherwise perhaps scan and import all widgets
// Attempt to import the component
let component = resolveComponent(
  `@/components/${props.widget.type}/${props.widget.component}.vue`,
)
</script>

<template>
  <div>
    <slot name="beforeComponent" />
    <Component :is="component" :widget="widget">
      <slot />
    </Component>
  </div>
  <slot name="afterComponent" />
</template>
