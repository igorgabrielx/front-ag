<script setup>
// Aqui você pode adicionar lógica reativa do Vue
import { ref, onUnmounted, onMounted, nextTick } from 'vue'
import axios from "axios"
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const populationSize = ref(100)
const generations = ref(400)
const crossoverRate = ref(0.65)
const mutationRate = ref(0.008)

const executionId = ref(null)
const executionStatus = ref(null)
const executionData = ref(null)
const pollingInterval = ref(null)

const chartCanvas = ref(null)
const fitnessChart = ref(null)

const startExecution = async () => {
  try {
    const response = await axios.post("http://localhost:5000/api/genetic/execute-async-ag", {
      pop_size: populationSize.value,
      num_gen: generations.value,
      taxa_crossover: crossoverRate.value,
      taxa_mutation: mutationRate.value
    })
    console.log("Execução iniciada:", response.data)
    
    executionId.value = response.data.execution_id
    executionStatus.value = "pendente"
    executionData.value = null
    
    // Iniciar polling
    startPolling()
  } catch (error) {
    console.error("Erro ao iniciar execução:", error)
    alert("Erro ao iniciar a execução. Verifique o console.")
    executionStatus.value = "erro"
  }
}

const startPolling = () => {
  // Limpar intervalo anterior se existir
  if (pollingInterval.value) {
    clearInterval(pollingInterval.value)
  }
  
  // Fazer primeira consulta imediatamente
  checkStatus()
  
  // Configurar polling a cada 2 segundos
  pollingInterval.value = setInterval(() => {
    checkStatus()
  }, 2000)
}

const checkStatus = async () => {
  if (!executionId.value) return
  
  try {
    const response = await axios.get(`http://localhost:5000/api/genetic/get-status-queue/${executionId.value}`)
    
    if (response.data && response.data.data) {
      executionData.value = response.data.data
      executionStatus.value = response.data.data.status
      
      // Parar polling se status for concluído ou erro
      if (executionStatus.value === "concluido" || executionStatus.value === "erro") {
        stopPolling()
        // Atualizar gráfico quando execução concluir
        if (executionStatus.value === "concluido") {
          loadFitnessChart()
        }
      }
    }
  } catch (error) {
    console.error("Erro ao verificar status:", error)
    stopPolling()
    executionStatus.value = "erro"
  }
}

const stopPolling = () => {
  if (pollingInterval.value) {
    clearInterval(pollingInterval.value)
    pollingInterval.value = null
  }
}

const loadFitnessChart = async () => {
  try {
    const response = await axios.get('http://127.0.0.1:5000/api/genetic/get-all-executions')
    
    if (response.data && response.data.data) {
      const executions = response.data.data
      
      // Extrair dados relevantes
      const labels = executions.map(exec => exec.num_gen)
      const values = executions.map(exec => exec.max_fitness)
      
      // Destruir gráfico anterior se existir
      if (fitnessChart.value) {
        fitnessChart.value.destroy()
      }
      
      // Aguardar o próximo tick para garantir que o canvas está renderizado
      await nextTick()
      
      if (chartCanvas.value) {
        const ctx = chartCanvas.value.getContext('2d')
        
        fitnessChart.value = new Chart(ctx, {
          type: 'line',
          data: {
            labels: labels,
            datasets: [{
              label: 'Melhor Fitness por Execução',
              data: values,
              borderWidth: 2,
              borderColor: '#136dec',
              backgroundColor: 'rgba(19, 109, 236, 0.2)',
              tension: 0.3,
              pointRadius: 4,
              pointHoverRadius: 6,
              pointBackgroundColor: '#136dec',
              pointBorderColor: '#ffffff',
              pointBorderWidth: 2
            }]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
              legend: { 
                display: true,
                labels: {
                  color: '#92a9c9',
                  font: {
                    family: 'Inter, sans-serif'
                  }
                }
              },
              title: {
                display: true,
                text: 'Evolução do Fitness Máximo x Número de Gerações',
                color: '#ffffff',
                font: {
                  family: 'Inter, sans-serif',
                  size: 14,
                  weight: 'bold'
                }
              }
            },
            scales: {
              x: {
                title: { 
                  display: true, 
                  text: 'Número de Gerações',
                  color: '#92a9c9',
                  font: {
                    family: 'Inter, sans-serif'
                  }
                },
                ticks: {
                  color: '#92a9c9',
                  font: {
                    family: 'Inter, sans-serif'
                  }
                },
                grid: {
                  color: 'rgba(50, 72, 103, 0.5)'
                }
              },
              y: {
                title: { 
                  display: true, 
                  text: 'Fitness Máximo',
                  color: '#92a9c9',
                  font: {
                    family: 'Inter, sans-serif'
                  }
                },
                ticks: {
                  color: '#92a9c9',
                  font: {
                    family: 'Inter, sans-serif'
                  }
                },
                grid: {
                  color: 'rgba(50, 72, 103, 0.5)'
                }
              }
            }
          }
        })
      }
    }
  } catch (error) {
    console.error('Erro ao carregar dados do gráfico:', error)
  }
}

// Carregar gráfico quando o componente for montado
onMounted(() => {
  loadFitnessChart()
})

// Limpar intervalo e gráfico quando o componente for desmontado
onUnmounted(() => {
  stopPolling()
  if (fitnessChart.value) {
    fitnessChart.value.destroy()
  }
})
</script>

<template>
  <div class="bg-[#101822] min-h-screen w-screen flex items-center justify-center p-4">
    <div class="w-full max-w-2xl rounded-xl border border-[#324867] bg-[#111822] p-6 md:p-8">
      
      <!-- Header -->
      <div class="mb-6 text-center">
        <h1 class="font-display text-3xl font-black tracking-[-0.033em] text-white">
          Configurador de Algoritmo Genético
        </h1>
        <p class="font-display mt-2 text-base font-normal leading-normal text-[#92a9c9]">
          Defina os parâmetros para iniciar a execução do algoritmo.
        </p>
      </div>
      
      <!-- Inputs -->
      <div class="space-y-6">
        <div class="grid grid-cols-1 gap-6 md:grid-cols-2">
          <label class="flex flex-col">
            <div class="flex items-center gap-2 pb-2">
              <p class="font-display text-base font-medium text-white">Tamanho da População</p>
              <div class="group relative flex cursor-pointer items-center">
                <span class="material-symbols-outlined text-base text-[#92a9c9]">help_outline</span>
                <div class="absolute bottom-full left-1/2 mb-2 w-max -translate-x-1/2 scale-0 transform rounded-lg bg-[#192433] px-3 py-2 text-center text-sm text-white transition-all group-hover:scale-100">
                  Número de indivíduos na população.
                </div>
              </div>
            </div>
            <input v-model="populationSize" type="number" placeholder="e.g., 100" 
              class="form-input font-display h-14 w-full rounded-lg border border-[#324867] bg-[#192433] p-[15px] text-base text-white placeholder:text-[#92a9c9] focus:border-primary focus:outline-0"/>
          </label>

          <label class="flex flex-col">
            <div class="flex items-center gap-2 pb-2">
              <p class="font-display text-base font-medium text-white">Número de Gerações</p>
              <div class="group relative flex cursor-pointer items-center">
                <span class="material-symbols-outlined text-base text-[#92a9c9]">help_outline</span>
                <div class="absolute bottom-full left-1/2 mb-2 w-max -translate-x-1/2 scale-0 transform rounded-lg bg-[#192433] px-3 py-2 text-center text-sm text-white transition-all group-hover:scale-100">
                  Número máximo de iterações do algoritmo.
                </div>
              </div>
            </div>
            <input v-model="generations" type="number" placeholder="e.g., 50" 
              class="form-input font-display h-14 w-full rounded-lg border border-[#324867] bg-[#192433] p-[15px] text-base text-white placeholder:text-[#92a9c9] focus:border-primary focus:outline-0"/>
          </label>
        </div>

        <!-- Crossover -->
        <div class="flex flex-col gap-3">
          <div class="flex justify-between items-center">
            <p class="font-display text-base font-medium text-white">Taxa de Crossover</p>
            <p class="font-display text-sm text-white">{{ crossoverRate.toFixed(2) }}</p>
          </div>
          <div class="relative py-2">
            <div class="h-1.5 w-full bg-[#324867] rounded-full relative">
              <div class="absolute h-full bg-primary rounded-full" :style="{ width: (crossoverRate * 100) + '%' }"></div>
            </div>
            <input 
              v-model.number="crossoverRate" 
              type="range" 
              min="0" 
              max="1" 
              step="0.01"
              class="absolute top-0 left-0 w-full h-full cursor-pointer slider-input"
            />
          </div>
        </div>

        <!-- Mutation -->
        <div class="flex flex-col gap-3">
          <div class="flex justify-between items-center">
            <p class="font-display text-base font-medium text-white">Taxa de Mutação</p>
            <p class="font-display text-sm text-white">{{ mutationRate.toFixed(3) }}</p>
          </div>
          <div class="relative py-2">
            <div class="h-1.5 w-full bg-[#324867] rounded-full relative">
              <div class="absolute h-full bg-primary rounded-full" :style="{ width: (mutationRate * 100) + '%' }"></div>
            </div>
            <input 
              v-model.number="mutationRate" 
              type="range" 
              min="0" 
              max="1" 
              step="0.001"
              class="absolute top-0 left-0 w-full h-full cursor-pointer slider-input"
            />
          </div>
        </div>
      </div>

      <!-- Button -->
      <div class="mt-8 flex justify-center">
        <button @click="startExecution" class="flex h-12 min-w-[84px] w-full cursor-pointer items-center justify-center rounded-lg bg-primary px-5 text-base font-bold text-white hover:bg-blue-600">
          <span class="material-symbols-outlined mr-2">play_arrow</span>
          Executar Algoritmo
        </button>
      </div>

      <!-- Status -->
      <div class="mt-8">
        <p class="font-display text-base font-medium text-white">Status da Execução</p>
        <div class="mt-2 flex min-h-[100px] flex-col items-center justify-center rounded-lg border border-dashed border-[#324867] bg-[#192433] p-4 text-center">
          <div v-if="!executionStatus" class="font-display text-sm text-[#92a9c9]">
            Aguardando configuração para iniciar a execução.
          </div>
          <div v-else class="w-full space-y-2">
            <div class="flex items-center justify-center gap-2">
              <span v-if="executionStatus === 'pendente'" class="material-symbols-outlined text-yellow-500 animate-spin">hourglass_empty</span>
              <span v-else-if="executionStatus === 'concluido'" class="material-symbols-outlined text-green-500">check_circle</span>
              <span v-else-if="executionStatus === 'erro'" class="material-symbols-outlined text-red-500">error</span>
              <span v-else class="material-symbols-outlined text-blue-500">sync</span>
              <p class="font-display text-sm font-medium text-white capitalize">
                Status: {{ executionStatus }}
              </p>
            </div>
            <div v-if="executionId" class="font-display text-xs text-[#92a9c9]">
              ID: {{ executionId }}
            </div>
            <div v-if="executionData && executionData.start_time" class="font-display text-xs text-[#92a9c9]">
              Iniciado em: {{ new Date(executionData.start_time).toLocaleString('pt-BR') }}
            </div>
            <div v-if="executionData && executionData.end_time" class="font-display text-xs text-[#92a9c9]">
              Finalizado em: {{ new Date(executionData.end_time).toLocaleString('pt-BR') }}
            </div>
          </div>
        </div>
      </div>
      
      <!-- Evolução do Fitness -->
      <div class="mt-8">
        <p class="font-display text-base font-medium text-white">Evolução do Fitness</p>
        <div class="mt-2 flex h-64 w-full flex-col justify-between rounded-lg border border-[#324867] bg-[#192433] p-4">
          <canvas ref="chartCanvas"></canvas>
        </div>
      </div>

      <!-- Resultado -->
      <div class="mt-8 space-y-4">
        <div class="flex flex-col">
          <p class="font-display text-base font-medium text-white">Resultado do Melhor Cromossomo</p>
          <div class="flex h-14 min-w-0 flex-1 items-center overflow-hidden rounded-lg border border-[#324867] bg-[#192433] px-4">
            <p class="font-display text-base text-[#92a9c9]">-</p>
          </div>
        </div>
        <div class="flex flex-col">
          <p class="font-display text-base font-medium text-white">Valor desse Cromossomo Aplicado</p>
          <div class="flex h-14 min-w-0 flex-1 items-center overflow-hidden rounded-lg border border-[#324867] bg-[#192433] px-4">
            <p class="font-display text-base text-[#92a9c9]">-</p>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<style>
html, body {
  font-family: 'Inter', sans-serif;
  background-color: #101822;
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

/* Estilos para os sliders */
.slider-input {
  -webkit-appearance: none;
  appearance: none;
  background: transparent;
  margin: 0;
  padding: 0;
}

.slider-input::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  background: #136dec;
  border-radius: 50%;
  cursor: pointer;
  margin-top: -8px;
  transition: all 0.2s;
  position: relative;
  z-index: 10;
}

.slider-input::-webkit-slider-thumb:hover {
  background: #0f5bc4;
  transform: scale(1.15);
}

.slider-input::-moz-range-thumb {
  width: 18px;
  height: 18px;
  background: #136dec;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s;
  position: relative;
  z-index: 10;
}

.slider-input::-moz-range-thumb:hover {
  background: #0f5bc4;
  transform: scale(1.15);
}
</style>
