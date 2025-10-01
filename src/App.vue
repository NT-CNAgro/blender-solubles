<template>
  <div class="h-screen overflow-hidden p-4 bg-gray-50">
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 h-full overflow-auto">
      <!-- Card izquierda: Entradas -->
      <!----------------------------->
      <article
        class="flex flex-col rounded-xl border bg-white shadow-sm overflow-auto"
      >
        <header
          class="sticky top-0 z-10 border-b bg-white/80 backdrop-blur p-3 rounded-t-xl"
        >
          <h2 class="text-sm font-semibold">Entradas de mezcla</h2>
        </header>

        <div class="min-h-0 grow overflow-y-auto p-4 space-y-4">
          <div class="flex items-center justify-between">
            <button
              class="px-3 py-2 text-sm rounded-lg border hover:bg-gray-50"
              @click="agregarEntrada()"
            >
              + Agregar componente
            </button>
            <small class="text-gray-500">Aportes definidos por 100 kg</small>
          </div>

          <div
            v-for="(e, idx) in entradas"
            :key="e.id"
            class="rounded-lg border p-3 space-y-3"
          >
            <div class="flex gap-3 items-end">
              <div class="grow">
                <label class="block text-xs font-medium">Producto</label>
                <select
                  v-model="e.productoKey"
                  class="mt-1 w-full border rounded p-2"
                >
                  <option disabled value="">Selecciona…</option>
                  <option
                    v-for="(def, key) in fertilizantes"
                    :key="key"
                    :value="key"
                  >
                    {{ def.nombre }}
                  </option>
                </select>
              </div>

              <div class="w-40">
                <label class="block text-xs font-medium">Cantidad (kg)</label>
                <input
                  type="number"
                  min="0"
                  step="any"
                  v-model.number="e.cantidad"
                  class="mt-1 w-full border rounded p-2"
                  placeholder="0"
                />
              </div>

              <button
                class="h-9 px-3 text-sm rounded-lg border hover:bg-red-50 hover:border-red-300"
                @click="eliminarEntrada(idx)"
                v-if="entradas.length > 1"
              >
                Eliminar
              </button>
            </div>

            <!-- Aportes por fila -->
            <!--------------------->
            <div
              v-if="e.productoKey"
              class="text-xs text-gray-700 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 overflow-auto"
            >
              <div v-for="col in columnas" :key="col.key">
                <span class="text-gray-500">{{ col.label }}:</span>
                <span class="ml-1 font-medium">{{
                  aporteFila(e)[col.key].toFixed(2)
                }}</span>
              </div>
            </div>
          </div>
        </div>
      </article>

      <!-- Card derecha: Totales -->
      <article class="flex flex-col rounded-xl border bg-white shadow-sm">
        <header
          class="sticky top-0 z-10 border-b bg-white/80 backdrop-blur p-3 rounded-t-xl"
        >
          <h2 class="text-sm font-semibold">Totales</h2>
        </header>

        <div class="min-h-0 grow overflow-y-auto p-4 space-y-4">
          <!-- Totales tabla -->
          <div class="overflow-auto rounded-lg border">
            <table class="min-w-full text-sm">
              <thead class="bg-gray-50">
                <tr>
                  <th
                    v-for="col in columnas"
                    :key="col.key"
                    class="px-3 py-2 text-left font-medium text-gray-600"
                  >
                    {{ col.label }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td v-for="col in columnas" :key="col.key" class="px-3 py-2">
                    {{ totales[col.key].toFixed(2) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Detalle por componente (opcional) -->
          <details class="rounded-lg border p-3">
            <summary class="cursor-pointer text-sm font-medium">
              Ver desglose por componente
            </summary>
            <div class="mt-3 space-y-3">
              <div
                v-for="(e, i) in entradas"
                :key="'det-' + e.id"
                class="rounded border p-2"
              >
                <div class="text-sm font-semibold">
                  {{ tituloEntrada(e, i) }}
                </div>
                <div
                  class="mt-2 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 text-xs"
                >
                  <div v-for="col in columnas" :key="col.key">
                    <span class="text-gray-500">{{ col.label }}:</span>
                    <span class="ml-1 font-medium">{{
                      aporteFila(e)[col.key].toFixed(2)
                    }}</span>
                  </div>
                </div>
              </div>
            </div>
          </details>
        </div>
      </article>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, computed } from "vue";

/**
 * Columnas estándar del balance
 */
const columnas = [
  { key: "N_NO3", label: "N-NO₃" },
  { key: "N_NH4", label: "N-NH₄" },
  { key: "N_URE", label: "N-Ureico" },
  { key: "N", label: "N" },
  { key: "P", label: "P" },
  { key: "K", label: "K" },
  { key: "Cl", label: "Cl" },
  { key: "S", label: "S" },
  { key: "MgO", label: "MgO" },
  { key: "CaO", label: "CaO" },
  { key: "B", label: "B" },
  { key: "Fe", label: "Fe" },
  { key: "Zn", label: "Zn" },
  { key: "Cu", label: "Cu" },
  { key: "Mn", label: "Mn" },
  { key: "Mo", label: "Mo" },
] as const;

type NutrKey = (typeof columnas)[number]["key"];

/**
 * Definición de fertilizantes (aportes por 100 kg)
 * Nota: puedes agregar más productos sin tocar la lógica.
 */
const fertilizantes: Record<
  string,
  {
    nombre: string;
    por100: Partial<Record<NutrKey, number>>;
  }
> = {
  // NITRATO DE CALCIO
  nitrato_calcio: {
    nombre: "Nitrato de calcio",
    por100: {
      N_NO3: 14.4,
      N_NH4: 1.1,
      CaO: 26.5,
    },
  },

  // NITRATO DE POTASIO
  nitrato_potasio: {
    nombre: "Nitrato de potasio",
    por100: {
      N_NO3: 13.0,
      K: 45.0, // desde K2O según tu dato
      Cl: 0.5,
    },
  },

  // Ácido Bórico
  acido_borico: {
    nombre: "Ácido Bórico",
    por100: {
      B: 17,
    },
  },

  // Fosfato monoamónico
  map: {
    nombre: "Fosfato Monoamónico",
    por100: {
      N_NH4: 12,
      P: 60,
    },
  },

  // Fosfato monopotásico
  fostato_monopotasico: {
    nombre: "Fosfato Monopotásico",
    por100: {
      P: 52,
      K: 34,
    },
  },

  // Muriato de potasio
  muriato_potasio: {
    nombre: "Muriato de potasio",
    por100: {
      K: 60, // desde K2O según tu dato
      Cl: 48,
    },
  },

  // Nitrato de magnesio
  nitrato_magnesio: {
    nombre: "Nitrato de magnesio",
    por100: {
      N_NO3: 11.5,
      MgO: 15.0,
    },
  },

  // Sulfato de Amonio
  sulfato_amonio: {
    nombre: "Sulfato de amonio",
    por100: {
      N_NH4: 21,
      S: 22,
    },
  },

  // Sulfato de Magnesio Heptahidratado
  sulfato_magnesio_hepta: {
    nombre: "Sulfato de magnesio heptahidratado",
    por100: {
      MgO: 16,
      S: 13,
    },
  },

  // Sulfato de Magnesio Anhidro
  sulfato_magnesio_anhidro: {
    nombre: "Sulfato de magnesio anhidro",
    por100: {
      MgO: 32,
      S: 26,
    },
  },

  // Sulfato de Manganeso
  sulfato_manganeso: {
    nombre: "Sulfato de manganeso",
    por100: {
      Mn: 32,
      S: 36,
    },
  },

  // Sulfato de potasio
  sulfato_potasio: {
    nombre: "Sulfato de potasio",
    por100: {
      K: 51, // desde K2O según tu dato
      S: 17.4,
    },
  },

  // Sulfato de Zinc Heptahidratado
  sulfato_zinc_hepta: {
    nombre: "Sulfato de zinc heptahidratado",
    por100: {
      Zn: 22,
      S: 19,
    },
  },

  // Fosfato de urea
  fosfato_urea: {
    nombre: "Fosfato de urea",
    por100: {
      N_URE: 16,
      P: 45,
    },
  },

  // Sulfonitrato de amonio 26%
  sulfonitrato_amonio: {
    nombre: "Sulfonitrato de amonio 26%",
    por100: {
      N_NO3: 7.5,
      N_NH4: 18.5,
      S: 13,
    },
  },

  // Urea granulada
  urea: {
    nombre: "Urea granulada",
    por100: {
      N_URE: 46,
    },
  },

  // Deltamicro Fe EDDHA 6% - 80% Orto-Orto
  deltamicro_fe_eddha: {
    nombre: "Deltamicro Fe EDDHA 6% - 80% Orto-Orto",
    por100: {
      Fe: 6,
    },
  },

  // Folikel Zn EDTA
  folikel_zn_edta: {
    nombre: "Folikel Zn EDTA",
    por100: {
      Zn: 15,
    },
  },

  // Sulfato de Cobre
  sulfato_cobre: {
    nombre: "Sulfato de cobre",
    por100: {
      Cu: 23,
    },
  },

  // Fertitec 21% + DMPP
  fertitec_dmpp: {
    nombre: "Fertitec 21% + DMPP",
    por100: {
      N_NH4: 4.5,
      S: 22,
    },
  },
};

/**
 * Entradas del usuario (producto + cantidad)
 */
let nextId = 1;
const entradas = reactive<
  { id: number; productoKey: string; cantidad: number }[]
>([
  // Demo inicial para replicar tu ejemplo 100 + 100
  { id: nextId++, productoKey: "nitrato_calcio", cantidad: 100 },
  { id: nextId++, productoKey: "nitrato_potasio", cantidad: 100 },
]);

function agregarEntrada() {
  entradas.push({ id: nextId++, productoKey: "", cantidad: 0 });
}
function eliminarEntrada(idx: number) {
  entradas.splice(idx, 1);
}

/**
 * Cálculo de aporte por fila
 */
function aporteFila(e: { productoKey: string; cantidad: number }) {
  const base: Record<NutrKey, number> = {
    N_NO3: 0,
    N_NH4: 0,
    N_URE: 0,
    N: 0,
    P: 0,
    K: 0,
    Cl: 0,
    S: 0,
    MgO: 0,
    CaO: 0,
    B: 0,
    Fe: 0,
    Zn: 0,
    Cu: 0,
    Mn: 0,
    Mo: 0,
  };
  const def = fertilizantes[e.productoKey];
  if (!def || !e.cantidad) return base;

  // Escala por cantidad real (los datos están "por 100 kg")
  const factor = e.cantidad / 100;

  for (const k of Object.keys(def.por100) as NutrKey[]) {
    base[k] = (def.por100[k] || 0) * factor;
  }

  // N total = suma de fracciones
  base.N = base.N_NO3 + base.N_NH4 + base.N_URE;
  return base;
}

/**
 * Totales agregados
 */
const totales = computed(() => {
  const acc: Record<NutrKey, number> = {
    N_NO3: 0,
    N_NH4: 0,
    N_URE: 0,
    N: 0,
    P: 0,
    K: 0,
    Cl: 0,
    S: 0,
    MgO: 0,
    CaO: 0,
    B: 0,
    Fe: 0,
    Zn: 0,
    Cu: 0,
    Mn: 0,
    Mo: 0,
  };
  for (const e of entradas) {
    const row = aporteFila(e);
    for (const k of Object.keys(acc) as NutrKey[]) {
      acc[k] += row[k];
    }
  }
  // Asegura coherencia de N total
  acc.N = acc.N_NO3 + acc.N_NH4 + acc.N_URE;
  return acc;
});

function tituloEntrada(
  e: { productoKey: string; cantidad: number },
  i: number
) {
  const name = fertilizantes[e.productoKey]?.nombre ?? `Componente ${i + 1}`;
  return `${name} — ${e.cantidad || 0} kg`;
}
</script>

<style>
html,
body,
#app {
  height: 100%;
}
</style>
