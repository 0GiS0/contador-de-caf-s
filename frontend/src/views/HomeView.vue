<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { api } from '../composables/useApi'
import { useRequirements } from '../composables/useRequirements'
import type { HealthStatus, Requirement } from '../types'
import RequirementCard from '../components/RequirementCard.vue'

const health = ref<HealthStatus | null>(null)
const healthLoading = ref(true)
const { requirements, loading, fetchRequirements, updateStatus } = useRequirements()

const doneCount = computed(() => requirements.value.filter((r) => r.status === 'DONE').length)
const progress = computed(() =>
  requirements.value.length ? Math.round((doneCount.value / requirements.value.length) * 100) : 0,
)

const coffeeCount = ref(0)

function addCoffee() {
  coffeeCount.value++
}

function resetCounter() {
  coffeeCount.value = 0
}

const showSplash = computed(() => {
  if (healthLoading.value || loading.value) return true
  if (doneCount.value === 0) return true
  return false
})

onMounted(async () => {
  try {
    const { data } = await api.get<HealthStatus>('/api/health')
    health.value = data
    await fetchRequirements()
  } catch {
    // Backend not running yet
  } finally {
    healthLoading.value = false
  }
})

const nextStatus: Record<string, Requirement['status']> = {
  PENDING: 'IN_PROGRESS',
  IN_PROGRESS: 'DONE',
  DONE: 'PENDING',
}

async function cycleStatus(req: Requirement) {
  await updateStatus(req.id, nextStatus[req.status])
}
</script>

<template>
  <div
    v-if="showSplash"
    class="flex min-h-[calc(100vh-73px)] flex-col items-center justify-center text-center"
  >
    <div class="mb-8">
      <span class="animate-pulse text-9xl">☕</span>
    </div>
    <h1 class="text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">
      Estamos construyendo tu idea
    </h1>
    <p class="mt-3 text-lg text-gray-500">
      ¿Momento de un café? Mientras tanto, aquí tienes tu lista de deseos:
    </p>
    <div
      v-if="health"
      class="mt-8 inline-flex items-center gap-2 rounded-full bg-green-50 px-4 py-2 text-sm text-green-700"
    >
      <span class="h-2 w-2 rounded-full bg-green-500"></span>
      Backend conectado — v{{ health.version }}
    </div>
    <div
      v-else
      class="mt-8 inline-flex items-center gap-2 rounded-full bg-amber-50 px-4 py-2 text-sm text-amber-700"
    >
      <span class="h-2 w-2 animate-pulse rounded-full bg-amber-500"></span>
      Conectando con el backend...
    </div>
    <div class="mx-auto mt-10 w-full max-w-md text-left">
      <h2 class="mb-3 text-sm font-semibold uppercase tracking-wide text-gray-400">
        📋 Requisitos del proyecto
      </h2>
      <div v-if="loading" class="py-4 text-center text-sm text-gray-400">
        Cargando requisitos...
      </div>
      <ul v-else class="space-y-2 opacity-75">
        <li
          v-for="req in requirements"
          :key="req.id"
          class="flex items-center gap-3 rounded-lg border border-dashed border-gray-300 bg-white/60 px-4 py-2.5 text-sm text-gray-500"
        >
          <span class="text-base">{{ req.status === 'DONE' ? '✅' : '⏳' }}</span>
          {{ req.title }}
        </li>
      </ul>
    </div>
  </div>

  <div
    v-else
    class="-mx-4 -my-8 flex min-h-[calc(100vh-73px)] flex-col items-center justify-center bg-amber-50 p-8 sm:-mx-6 lg:-mx-8"
  >
    <div class="mb-4 text-8xl">☕</div>
    <h1 class="mb-8 text-3xl font-bold text-amber-900">Contador de Cafés</h1>
    <div class="mb-8 rounded-3xl border-4 border-amber-800 bg-amber-100 p-12 shadow-lg">
      <span class="text-9xl font-bold text-amber-900">{{ coffeeCount }}</span>
    </div>
    <p class="mb-8 text-xl text-amber-700">cafés hoy</p>
    <div class="flex gap-6">
      <button
        class="h-20 w-20 rounded-full bg-amber-800 text-4xl font-bold text-white shadow-lg transition-all duration-200 hover:scale-110 hover:bg-amber-900 active:scale-95"
        @click="addCoffee"
      >
        +
      </button>
      <button
        class="rounded-full bg-amber-600 px-6 py-4 text-lg font-semibold text-white shadow-lg transition-all duration-200 hover:scale-105 hover:bg-amber-700 active:scale-95"
        @click="resetCounter"
      >
        Reset
      </button>
    </div>
    <p class="mt-12 text-sm text-amber-600">¡No te pases con la cafeína! 😄</p>
    <div class="hidden">
      <span>{{ progress }}</span>
      <RequirementCard
        v-for="r in requirements"
        :key="r.id"
        :requirement="r"
        @cycle-status="cycleStatus"
      />
    </div>
  </div>
</template>

<style scoped>
@keyframes steam {
  0% {
    opacity: 0;
    transform: translateY(0) scaleX(1);
  }
  50% {
    opacity: 0.6;
  }
  100% {
    opacity: 0;
    transform: translateY(-20px) scaleX(1.5);
  }
}

.animate-steam-1 {
  animation: steam 2s ease-in-out infinite;
}
.animate-steam-2 {
  animation: steam 2s ease-in-out 0.4s infinite;
}
.animate-steam-3 {
  animation: steam 2s ease-in-out 0.8s infinite;
}
</style>
