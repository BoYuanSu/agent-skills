<script setup lang="ts">
interface ExampleOutputProps {}

const props = withDefaults(defineProps<ExampleOutputProps>(), {});
</script>

<template>
  <article class="ComponentName">
    <header class="ComponentName__header">
      <img v-if="iconUrl" class="ComponentName__icon" :src="iconUrl" alt="" />
      <h3 class="ComponentName__title">{{ title }}</h3>
      <p v-if="subtitle" class="ComponentName__subtitle">{{ subtitle }}</p>
    </header>

    <section class="ComponentName__content">
      <ul v-if="items?.length" class="ComponentName__list">
        <li v-for="(item, index) in items" :key="item.id ?? index" class="ComponentName__listItem">
          {{ item.label }}
        </li>
      </ul>
    </section>

    <footer class="ComponentName__footer">
      <button class="ComponentName__action">Action</button>
    </footer>
  </article>
</template>

<style lang="scss" scoped>
.ComponentName {
  .ComponentName__header {
  }
  .ComponentName__icon {
  }
  .ComponentName__title {
  }
  .ComponentName__subtitle {
  }
  .ComponentName__content {
  }
  .ComponentName__list {
  }
  .ComponentName__listItem {
  }
  .ComponentName__footer {
  }
  .ComponentName__action {
  }
}
</style>
