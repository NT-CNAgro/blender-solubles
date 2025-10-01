<template>
  <article class="flex flex-col rounded-xl border bg-white shadow-sm">
    <!-- Header -->
    <header
      class="sticky top-0 z-10 border-b bg-white/80 backdrop-blur p-3 rounded-t-xl"
    >
      <h2 class="text-sm font-semibold">Nitrato de Calcio</h2>
    </header>

    <!-- Formulario -->
    <div class="min-h-0 grow overflow-y-auto p-4 space-y-4">
      <div>
        <label class="block text-sm font-medium">Cantidad (kg)</label>
        <input
          type="number"
          v-model.number="cantidad"
          class="mt-1 w-full border rounded p-2"
          placeholder="Ej: 100"
        />
      </div>

      <div class="space-y-2 text-sm">
        <p>
          N-NO₃:
          <span class="font-semibold">{{ aporteN_NO3.toFixed(2) }}</span> kg
        </p>
        <p>
          N-NH₄:
          <span class="font-semibold">{{ aporteN_NH4.toFixed(2) }}</span> kg
        </p>
        <p>
          Total N: <span class="font-semibold">{{ totalN.toFixed(2) }}</span> kg
        </p>
        <p>
          Calcio (Ca):
          <span class="font-semibold">{{ aporteCa.toFixed(2) }}</span> kg
        </p>
      </div>
    </div>
  </article>
</template>

<script setup>
import { ref, computed } from "vue";

// Cantidad ingresada por el usuario (kg)
const cantidad = ref(0);

// Factores de aporte (por 100 kg de producto)
const factorNO3 = 14.4 / 100;
const factorNH4 = 1.1 / 100;
const factorCa = 26.5 / 100;

// Computed para calcular aportes
const aporteN_NO3 = computed(() => cantidad.value * factorNO3);
const aporteN_NH4 = computed(() => cantidad.value * factorNH4);
const aporteCa = computed(() => cantidad.value * factorCa);
const totalN = computed(() => aporteN_NO3.value + aporteN_NH4.value);
</script>
