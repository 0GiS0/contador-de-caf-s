<script setup lang="ts">
import { RouterLink } from 'vue-router'
</script>

<template>
  <header class="border-b border-amber-200 bg-amber-100">
    <div class="mx-auto flex max-w-7xl items-center justify-between px-4 py-4 sm:px-6 lg:px-8">
      <RouterLink to="/" class="flex items-center gap-2 text-xl font-bold text-amber-900">
        <span>☕</span>
        <span>Contador de Cafés</span>
      </RouterLink>
      <nav class="flex items-center gap-6">
        <RouterLink
          to="/"
          class="text-sm font-medium text-amber-700 transition-colors hover:text-amber-900"
        >
          Inicio
        </RouterLink>
      </nav>
    </div>
  </header>
</template>
