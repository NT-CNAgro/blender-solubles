<template>
  <div class="h-screen overflow-hidden p-4 bg-gray-50 dark:bg-gray-900">
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 h-full overflow-auto">
      <!-- Card izquierda: Entradas -->
      <!----------------------------->
      <article class="flex flex-col rounded-xl border bg-white dark:bg-gray-800 shadow-sm md:col-span-1">
        <header class="sticky top-0 z-10 border-b bg-white/80 dark:bg-gray-800/80 backdrop-blur p-3 rounded-t-xl">
          <h2 class="text-sm font-semibold">Entradas de mezcla</h2>
        </header>

        <div class="min-h-0 grow overflow-y-auto p-4 space-y-4">
          <div class="flex items-center justify-between">
            <button class="px-3 py-2 text-sm rounded-lg border hover:bg-gray-50" @click="agregarEntrada()">
              + Agregar componente
            </button>
            <small class="text-gray-500">Aportes definidos por 100 kg</small>
          </div>

          <div v-for="(e, idx) in entradas" :key="e.id" class="rounded-lg border p-3 space-y-3">
            <div class="flex gap-3 items-end">
              <div class="grow">
                <label class="block text-xs font-medium">Producto</label>
                <select v-model="e.productoKey" class="mt-1 w-full border border-gray-300 dark:border-gray-600 rounded p-2
               bg-white dark:bg-gray-800 text-gray-900 dark:text-gray-100">
                  <option disabled value="">Selecciona…</option>
                  <option
                    v-for="([key, def]) in Object.entries(fertilizantes).sort((a, b) => a[1].nombre.localeCompare(b[1].nombre))"
                    :key="key" :value="key">
                    {{ def.nombre }}
                  </option>
                </select>
              </div>

              <div class="w-40">
                <label class="block text-xs font-medium">Cantidad (kg)</label>
                <input type="number" min="0" step="any" v-model.number="e.cantidad" class="mt-1 w-full border border-gray-300 dark:border-gray-600 rounded p-2
               bg-white dark:bg-gray-800 text-gray-900 dark:text-gray-100" placeholder="0" />
              </div>

              <button class="h-9 px-3 text-sm rounded-lg border hover:bg-red-50 hover:border-red-300"
                @click="eliminarEntrada(idx)" v-if="entradas.length > 1">
                Eliminar
              </button>
            </div>

            <!-- Aportes por fila -->
            <!--------------------->
            <div v-if="e.productoKey"
              class="text-xs text-gray-700 dark:text-gray-200 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 overflow-auto">
              <div v-for="col in columnas" :key="col.key">
                <span class="text-gray-500 dark:text-gray-400">{{ col.label }}:</span>
                <span class="ml-1 font-medium">{{
                  aporteFila(e)[col.key].toFixed(2)
                }}</span>
              </div>
            </div>
          </div>
        </div>
      </article>

      <!-- Card derecha: Totales -->
      <article class="flex flex-col rounded-xl border bg-white dark:bg-gray-800 shadow-sm md:col-span-2">
        <header class="sticky top-0 z-10 border-b bg-white/80 dark:bg-gray-800/80 backdrop-blur p-3 rounded-t-xl">
          <h2 class="text-sm font-semibold">Totales</h2>
        </header>

        <div class="min-h-0 grow overflow-y-auto p-4 space-y-4">
          <!-- Totales tabla -->
          <div class="overflow-auto rounded-lg border">
            <table class="min-w-full text-sm">
              <thead class="bg-gray-50 dark:bg-gray-800">
                <tr class="bg-white dark:bg-gray-800">
                  <th v-for="col in columnas" :key="col.key"
                    class="px-3 py-2 text-left font-medium text-gray-600 dark:text-gray-300">
                    {{ col.label }}
                  </th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200 dark:divide-gray-700">
                <!-- Fila 1: Aportes netos -->
                <tr class="bg-gray-50 dark:bg-gray-700 text-gray-700 dark:text-gray-200">
                  <td v-for="col in columnas" :key="col.key" class="px-3 py-2">
                    {{ totales[col.key].toFixed(2) }}
                  </td>
                </tr>

                <!-- Fila 2: Porcentajes -->
                <!-- <tr class="bg-gray-50 dark:bg-gray-700 text-gray-700 dark:text-gray-200">
                  <td v-for="col in columnas" :key="col.key" class="px-3 py-2">
                    {{ porcentajes[col.key].toFixed(2) }}%
                  </td>
                </tr> -->
              </tbody>
            </table>
          </div>

          <!-- Detalle por componente (opcional) -->
          <details class="rounded-lg border border-gray-200 dark:border-gray-700 p-3">
            <summary class="cursor-pointer text-sm font-medium">
              Ver desglose por componente
            </summary>
            <div class="mt-3 space-y-3">
              <div v-for="(e, i) in entradas" :key="'det-' + e.id" class="rounded border p-2">
                <div class="text-sm font-semibold">
                  {{ tituloEntrada(e, i) }}
                </div>
                <div class="mt-2 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 text-xs">
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


          <details class="rounded-lg border border-gray-200 dark:border-gray-700 p-3">
            <summary class="cursor-pointer text-sm font-medium">
              Datos Teóricos
            </summary>

            <div class="mt-3 space-y-3">
              <!-- pH -->
              <div class="rounded border p-2">
                <div class="text-sm font-semibold">pH estimado</div>
                <div class="mt-2 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 text-xs">
                  <div class="col-span-2 md:col-span-1">
                    <span class="text-gray-500">Valor:</span>
                    <span class="ml-1 font-medium">{{ phMezcla.toFixed(2) }} <span class="font-normal text-gray-600">
                        [1g/L]</span></span>
                  </div>

                </div>
              </div>

              <!-- CE -->
              <div class="rounded border p-2">
                <div class="text-sm font-semibold">Conductividad eléctrica</div>
                <div class="mt-2 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 text-xs">
                  <div class="col-span-2 md:col-span-1"><span class="font-normal text-gray-600"></span>
                    <span class="text-gray-500">Valor:</span>
                    <span class="ml-1 font-medium">
                      {{ ceMezcla.toFixed(2) }} <span class="font-normal text-gray-600">dS/m [1g/L]</span>
                    </span>
                  </div>

                </div>
              </div>

              <!-- Solubilidad -->
              <div class="rounded border p-2">
                <div class="text-sm font-semibold">Solubilidad a 20 °C</div>
                <div class="mt-2 grid grid-cols-2 md:grid-cols-5 gap-x-4 gap-y-2 text-xs">
                  <div class="col-span-2 md:col-span-1">
                    <span class="text-gray-500">Valor:</span>
                    <span class="ml-1 font-medium">
                      {{ solMezcla.toFixed(0) }} <span class="font-normal text-gray-600">g/L</span>
                    </span>
                  </div>

                </div>
              </div>
            </div>
          </details>
          <!-- Nuevo: Porcentajes ajustados (multiplicados por factor de columna) -->
          <details class="rounded-lg border border-gray-200 dark:border-gray-700 p-3">
            <summary class="cursor-pointer text-sm font-medium">
              Unidades de fertilizantes
            </summary>

            <div class="mt-3 overflow-auto rounded-lg border">
              <table class="min-w-full text-sm">
                <thead class="bg-gray-50 dark:bg-gray-800">
                  <tr class="bg-white dark:bg-gray-800">
                    <th v-for="col in columnas" :key="col.key"
                      class="px-3 py-2 text-left font-medium text-gray-600 dark:text-gray-300">
                      {{ col.label2 }}
                    </th>
                  </tr>
                </thead>

                <tbody class="divide-y divide-gray-200 dark:divide-gray-700">
                  <tr class="bg-gray-50 dark:bg-gray-700 text-gray-700 dark:text-gray-200">
                    <td v-for="col in columnas" :key="col.key" class="px-3 py-2">
                      {{ (porcentajesFactor[col.key] ?? 0).toFixed(3) }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </details>

          <!-- Relacion npk -->
          <!-- Reemplaza el contenido interno del detalle "Relaciones N-P-K" por esto -->
          <details class="rounded-lg border border-gray-200 dark:border-gray-700 p-3">
            <summary class="cursor-pointer text-sm font-medium">
              Relaciones N-P-K
            </summary>

            <div class="mt-3 space-y-3">
              <div class="rounded border p-2">
                <div class="text-sm font-semibold">N / P₂O₅</div>
                <div class="mt-2 text-xs">
                  <span class="text-gray-500">Valor:</span>
                  <span class="ml-1 font-medium">{{ relacionNPStr }}</span>
                </div>
              </div>

              <div class="rounded border p-2">
                <div class="text-sm font-semibold">N / K₂O</div>
                <div class="mt-2 text-xs">
                  <span class="text-gray-500">Valor:</span>
                  <span class="ml-1 font-medium">{{ relacionNKStr }}</span>
                </div>
              </div>

              <div class="rounded border p-2">
                <div class="text-sm font-semibold">K₂O / MgO</div>
                <div class="mt-2 text-xs">
                  <span class="text-gray-500">Valor:</span>
                  <span class="ml-1 font-medium">{{ relacionKMgStr }}</span>
                </div>
              </div>

              <div class="rounded border p-2">
                <div class="text-sm font-semibold">CaO / MgO</div>
                <div class="mt-2 text-xs">
                  <span class="text-gray-500">Valor:</span>
                  <span class="ml-1 font-medium">{{ relacionCaMgStr }}</span>
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
const totalCantidad = computed(() =>
  entradas.reduce((acc, e) => acc + (e.cantidad || 0), 0)
);

const porcentajes = computed(() => {
  const result: Record<NutrKey, number> = {} as any;
  for (const k of Object.keys(totales.value) as NutrKey[]) {
    // evita división por cero
    result[k] =
      totalCantidad.value > 0
        ? (totales.value[k] / totalCantidad.value) * 100
        : 0;
  }
  return result;
});

// nuevo: porcentajes multiplicados por el factor de cada columna
const porcentajesFactor = computed(() => {
  const result: Record<NutrKey, number> = {} as any;
  for (const col of columnas) {
    const key = col.key;
    const value = totales.value[key] ?? 0; // valor absoluto (misma unidad que totales)
    result[key] = value * (col.factor ?? 1);
  }
  return result;
});


/**
 * Columnas estándar del balance
 */
const columnas = [
  { key: "N_NO3", label: "N-NO₃", factor: 0.2259, label2: "N-NO₃" },
  { key: "N_NH4", label: "N-NH₄", factor: 1, label2: "N-NH₄" },
  { key: "N_URE", label: "N-Ureico", factor: 1, label2: "N-Ureico" },
  { key: "N", label: "N", factor: 1, label2: "N" },
  { key: "P", label: "P₂O₅", factor: 0.4364, label2: "P" },
  { key: "K", label: "K₂O", factor: 0.8302, label2: "K" },
  { key: "Cl", label: "Cl", factor: 1, label2: "Cl" },
  { key: "S", label: "S", factor: 1, label2: "S" },
  { key: "MgO", label: "MgO", factor: 0.603, label2: "Mg" },
  { key: "CaO", label: "CaO", factor: 0.714, label2: "Ca" },
  { key: "B", label: "B", factor: 1, label2: "B" },
  { key: "Fe", label: "Fe", factor: 1, label2: "Fe" },
  { key: "Zn", label: "Zn", factor: 1, label2: "Zn" },
  { key: "Cu", label: "Cu", factor: 1, label2: "Cu" },
  { key: "Mn", label: "Mn", factor: 1, label2: "Mn" },
  { key: "Mo", label: "Mo", factor: 1, label2: "Mo" },
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
    ficha: {
      ph: number;
      sol: number; // g/L @20°C
      Ce: number; // dS/m @1g/L
    };
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
    ficha: {
      ph: 5.09,
      sol: 1000,
      Ce: 1.16,
    },

  },

  // NITRATO DE POTASIO
  nitrato_potasio: {
    nombre: "Nitrato de potasio",
    por100: {
      N_NO3: 13.0,
      K: 45.0, // desde K2O según tu dato
    },
    ficha: {
      ph: 7.1,
      sol: 379,
      Ce: 1.31,
    },
  },

  // Ácido Bórico
  acido_borico: {
    nombre: "Ácido Bórico",
    por100: {
      B: 17,
    },
    ficha: {
      ph: 4.3,
      sol: 75,
      Ce: 0,
    },
  },

  // Fosfato monoamónico
  map: {
    nombre: "Fosfato Monoamónico",
    por100: {
      N_NH4: 12,
      P: 60,
    },
    ficha: {
      ph: 4.9,
      sol: 332,
      Ce: 0.83,
    },
  },

  // Fosfato monopotásico
  fostato_monopotasico: {
    nombre: "Fosfato Monopotásico",
    por100: {
      P: 52,
      K: 34,
    },
    ficha: {
      ph: 4.1,
      sol: 248,
      Ce: 0.76,
    },
  },

  // Muriato de potasio
  muriato_potasio: {
    nombre: "Muriato de potasio",
    por100: {
      K: 60, // desde K2O según tu dato
      Cl: 48,
    },
    ficha: {
      ph: 5.7,
      sol: 352,
      Ce: 1.79,
    },
  },

  // Nitrato de magnesio
  nitrato_magnesio: {
    nombre: "Nitrato de magnesio",
    por100: {
      N_NO3: 11.5,
      MgO: 15.0,
    },
    ficha: {
      ph: 5.56,
      sol: 1000,
      Ce: 0.85,
    },
  },

  // Sulfato de Amonio
  sulfato_amonio: {
    nombre: "Sulfato de amonio",
    por100: {
      N_NH4: 21,
      S: 22,
    },
    ficha: {
      ph: 5.5,
      sol: 723,
      Ce: 1.91,
    },
  },

  // Sulfato de Magnesio Heptahidratado
  sulfato_magnesio_hepta: {
    nombre: "Sulfato de magnesio heptahidratado",
    por100: {
      MgO: 16,
      S: 13,
    },
    ficha: {
      ph: 4.02,
      sol: 540,
      Ce: 0.73,
    },
  },

  // Sulfato de Magnesio Anhidro
  sulfato_magnesio_anhidro: {
    nombre: "Sulfato de magnesio anhidro",
    por100: {
      MgO: 32,
      S: 26,
    },
    ficha: {
      ph: 5.73,
      sol: 430,
      Ce: 1.28,
    },
  },

  // Sulfato de Manganeso
  sulfato_manganeso: {
    nombre: "Sulfato de manganeso",
    por100: {
      Mn: 32,
      S: 36,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
    },
  },

  // Sulfato de potasio
  sulfato_potasio: {
    nombre: "Sulfato de potasio",
    por100: {
      K: 51, // desde K2O según tu dato
      S: 17.4,
    },
    ficha: {
      ph: 7,
      sol: 184,
      Ce: 1.47,
    },
  },

  // Sulfato de Zinc Heptahidratado
  sulfato_zinc_hepta: {
    nombre: "Sulfato de zinc heptahidratado",
    por100: {
      Zn: 22,
      S: 19,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
    },
  },

  // Fosfato de urea
  fosfato_urea: {
    nombre: "Fosfato de urea",
    por100: {
      N_URE: 16,
      P: 45,
    },
    ficha: {
      ph: 1.8,
      sol: 300,
      Ce: 1.51,
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
    ficha: {
      ph: 5.38,
      sol: 714,
      Ce: 1.7,
    },
  },

  // Urea granulada
  urea: {
    nombre: "Urea granulada",
    por100: {
      N_URE: 46,
    },
    ficha: {
      ph: 5.8,
      sol: 1080,
      Ce: 0.02,
    },
  },

  // Deltamicro Fe EDDHA 6% - 80% Orto-Orto
  deltamicro_fe_eddha: {
    nombre: "Deltamicro Fe EDDHA 6% - 80% Orto-Orto",
    por100: {
      Fe: 6,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
    },
  },

  // Folikel Zn EDTA
  folikel_zn_edta: {
    nombre: "Folikel Zn EDTA",
    por100: {
      Zn: 15,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
    },
  },

  // Sulfato de Cobre
  sulfato_cobre: {
    nombre: "Sulfato de cobre",
    por100: {
      Cu: 23,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
    },
  },

  // Fertitec 21% + DMPP
  fertitec_dmpp: {
    nombre: "Fertitec 21% + DMPP",
    por100: {
      N_NH4: 4.5,
      S: 22,
    },
    ficha: {
      ph: 5.5,
      sol: 723,
      Ce: 1.91,
    },
  },

  // Actine
  Actine: {
    nombre: "Actine 46% + NPBT",
    por100: {
      N_URE: 46,
      Fe: 6,
    },
    ficha: {
      ph: 5.8,
      sol: 1080,
      Ce: 0.015,
    },
  },



  // NAscent nitro 25
  Nascent_nitro: {
    nombre: "Nascent Nitro 25%",
    por100: {
      N_NO3: 8.1,
      N_NH4: 16.7,
      S: 13,
    },
    ficha: {
      ph: 5.38,
      sol: 714,
      Ce: 1.7,
    },
  },


  // Nitrato de Amonio
  Nitrato_Amonio: {
    nombre: "Nitrato de Amonio",
    por100: {
      N_NO3: 17,
      N_NH4: 17,
    },
    ficha: {
      ph: 5.20,
      sol: 700,
      Ce: 1.6,
    },
  },

  // Sulfato de Fierro
  Sulfato_Fierro: {
    nombre: "Sulfato de Fierro",
    por100: {
      Fe: 19,
    },
    ficha: {
      ph: 0,
      sol: 0,
      Ce: 0,
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


const objetivoKg = 1000
const factorEscala = computed(() =>
  totalCantidad.value > 0 ? objetivoKg / totalCantidad.value : 0
)

type Proporcion = { id: number; nombre: string; kg_1t: number; porcentaje: number }

const proporciones1t = computed<Proporcion[]>(() => {
  const f = factorEscala.value
  return entradas.map((e, i) => {
    const nombre = fertilizantes[e.productoKey]?.nombre ?? `Componente ${i + 1}`
    const kg1t = (e.cantidad || 0) * f
    const pct = totalCantidad.value > 0 ? ((e.cantidad || 0) / totalCantidad.value) * 100 : 0
    return { id: e.id, nombre, kg_1t: kg1t, porcentaje: pct }
  })
})

const phMezcla = computed(() => {
  // suma ponderada: (pH_i * kg_1t_i)
  const sumatoria = proporciones1t.value.reduce((acc, e) => {
    const ph = fertilizantes[entradas.find(x => x.id === e.id)?.productoKey || '']?.ficha?.ph
    if (ph && e.kg_1t) acc += ph * e.kg_1t
    return acc
  }, 0)
  return sumatoria > 0 ? sumatoria / 1000 : 0
})

function tituloEntrada(
  e: { productoKey: string; cantidad: number },
  i: number
) {
  const name = fertilizantes[e.productoKey]?.nombre ?? `Componente ${i + 1}`;
  return `${name} — ${e.cantidad || 0} kg`;
}

const ceMezcla = computed(() => {
  // suma ponderada: (CE_i * kg_1t_i) / 1000
  const sumatoria = proporciones1t.value.reduce((acc, row) => {
    const entrada = entradas.find(e => e.id === row.id)
    const def = entrada ? fertilizantes[entrada.productoKey] : undefined
    const ce = def?.ficha?.Ce
    if (typeof ce === 'number' && row.kg_1t) {
      acc += ce * row.kg_1t
    }
    return acc
  }, 0)
  return sumatoria > 0 ? sumatoria / 1000 : 0
})


const solMezcla = computed(() => {
  const sumatoria = proporciones1t.value.reduce((acc, row) => {
    const entrada = entradas.find(e => e.id === row.id)
    const def = entrada ? fertilizantes[entrada.productoKey] : undefined
    const sol = def?.ficha?.sol
    if (typeof sol === 'number' && row.kg_1t) {
      acc += sol * row.kg_1t
    }
    return acc
  }, 0)
  return sumatoria > 0 ? sumatoria / 1000 : 0  // g/L @20°C
})


// helper: mapa de factores desde `columnas`
const factorMap = Object.fromEntries(columnas.map(c => [c.key, c.factor ?? 1])) as Record<string, number>;

// obtiene valor elemental (totales * factor de la columna)
function elemento(key: keyof typeof factorMap) {
  const raw = (totales.value as any)[key] ?? 0;
  return raw;
}

// relaciones (usando valores elementales)
const relacionNP = computed(() => {
  const N = elemento("N");
  const P = elemento("P");
  return P > 0 ? P / N : NaN;
});
const relacionNK = computed(() => {
  const N = elemento("N");
  const K = elemento("K");
  return K > 0 ? K / N : NaN;
});
const relacionKMg = computed(() => {
  const K = elemento("K");
  const Mg = elemento("MgO");
  return Mg > 0 ? K / Mg : NaN;
});
const relacionCaMg = computed(() => {
  const Ca = elemento("CaO");
  const Mg = elemento("MgO");
  return Mg > 0 ? Ca / Mg : NaN;
});

// strings formateadas para plantilla
const relacionNPStr = computed(() => Number.isFinite(relacionNP.value) ? relacionNP.value.toFixed(2) : "—");
const relacionNKStr = computed(() => Number.isFinite(relacionNK.value) ? relacionNK.value.toFixed(2) : "—");
const relacionKMgStr = computed(() => Number.isFinite(relacionKMg.value) ? relacionKMg.value.toFixed(2) : "—");
const relacionCaMgStr = computed(() => Number.isFinite(relacionCaMg.value) ? relacionCaMg.value.toFixed(2) : "—");



</script>

<style>
html,
body,
#app {
  height: 100%;
}
</style>
