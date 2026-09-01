<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-efrain', [])

const tiposReparacion = [
  'Cambio de pantalla', 'Cambio de batería', 'Cambio de pin de carga', 
  'Liberación', 'Mantenimiento de software', 'Cambio de flex', 'Otros'
]
const tecnicos = ['Efraín', 'Técnico 1', 'Técnico 2']
const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const estadosPago = ['Pagado', 'Pendiente', 'Abono']
const estadosEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

const isModalOpen = ref(false)
const isEditing = ref(false)
const currentId = ref(null)

const form = ref({
  cliente: '',
  equipo: '',
  reparacion: '',
  tecnico: '',
  fechaRecepcion: '',
  precio: '',
  metodoPago: '',
  estadoPago: '',
  estadoEquipo: 'Recibido',
  calificacion: 0,
  observaciones: ''
})

const resetForm = () => {
  form.value = {
    cliente: '',
    equipo: '',
    reparacion: '',
    tecnico: '',
    fechaRecepcion: '',
    precio: '',
    metodoPago: '',
    estadoPago: '',
    estadoEquipo: 'Recibido',
    calificacion: 0,
    observaciones: ''
  }
  isEditing.value = false
  currentId.value = null
}

const openModal = () => {
  resetForm()
  isModalOpen.value = true
}

const editServicio = (servicio) => {
  form.value = { ...servicio }
  isEditing.value = true
  currentId.value = servicio.id
  isModalOpen.value = true
}

const saveServicio = () => {
  if (isEditing.value) {
    let index = -1
    for(let i = 0; i < servicios.value.length; i++) {
      if(servicios.value[i].id === currentId.value) {
        index = i
      }
    }
    
    if (index !== -1) {
      servicios.value[index] = { ...form.value, id: currentId.value }
    }
  } else {
    servicios.value.push({ ...form.value, id: Date.now() })
  }
  isModalOpen.value = false
}

const confirmDeleteId = ref(null)
const isDeleteModalOpen = ref(false)

const askDelete = (id) => {
  confirmDeleteId.value = id
  isDeleteModalOpen.value = true
}

const deleteServicio = () => {
  let nuevosServicios = []
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].id !== confirmDeleteId.value) {
      nuevosServicios.push(servicios.value[i])
    }
  }
  servicios.value = nuevosServicios
  isDeleteModalOpen.value = false
}

const getPaymentColor = (estado) => {
  if (estado === 'Pagado') return 'positive'
  if (estado === 'Pendiente') return 'negative'
  if (estado === 'Abono') return 'warning'
  return 'grey'
}

const getEquipmentIcon = (estado) => {
  if (estado === 'Recibido') return 'inbox'
  if (estado === 'En reparación') return 'build'
  if (estado === 'Listo para entregar') return 'done'
  if (estado === 'Entregado') return 'check_circle'
  return 'devices'
}

const getEquipmentColor = (estado) => {
  if (estado === 'Recibido') return 'primary'
  if (estado === 'En reparación') return 'orange'
  if (estado === 'Listo para entregar') return 'teal'
  if (estado === 'Entregado') return 'positive'
  return 'grey'
}

const requiredRule = val => !!val || 'Este campo es requerido'
const priceRule = val => (val !== null && val !== '') || 'Debe ingresar un precio'

</script>

<template>
  <q-layout view="hHh lpR fFf" class="bg-grey-2">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title>
          📱 Taller Don Efraín
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        
        <div v-if="servicios.length === 0" class="text-center q-mt-xl text-grey-7">
          <q-icon name="devices_other" size="5rem" />
          <h5 class="q-mt-sm">No hay servicios registrados</h5>
          <p>Toca el botón "+" para registrar el primer equipo.</p>
        </div>

        <div class="row q-col-gutter-md">
          <div class="col-12 col-md-6 col-lg-4" v-for="servicio in servicios" :key="servicio.id">
            <q-card bordered class="q-mb-md" :class="{'bg-red-1': (servicio.estadoPago === 'Pendiente' || servicio.estadoPago === 'Abono') && servicio.estadoEquipo === 'Entregado'}">
              <q-card-section>
                <div class="text-h6">{{ servicio.cliente }}</div>
                <div class="text-subtitle2 text-grey-8">{{ servicio.equipo }}</div>
              </q-card-section>

              <q-separator />

              <q-card-section>
                <div class="row items-center q-mb-sm">
                  <q-icon name="build" class="q-mr-sm" color="primary" />
                  <span>{{ servicio.reparacion }}</span>
                </div>
                
                <div class="row items-center q-mb-sm">
                  <q-icon name="person" class="q-mr-sm" color="secondary" />
                  <span>Técnico: {{ servicio.tecnico }}</span>
                </div>

                <div class="row items-center q-mb-sm">
                  <q-icon name="event" class="q-mr-sm" color="grey-7" />
                  <span>Fecha: {{ servicio.fechaRecepcion }}</span>
                </div>

                <div class="q-mt-md q-gutter-sm">
                  <q-chip outline :color="getPaymentColor(servicio.estadoPago)">
                    Pago: {{ servicio.estadoPago }} (${{ servicio.precio }} - {{ servicio.metodoPago }})
                  </q-chip>
                  
                  <q-chip :color="getEquipmentColor(servicio.estadoEquipo)" text-color="white" :icon="getEquipmentIcon(servicio.estadoEquipo)">
                    {{ servicio.estadoEquipo }}
                  </q-chip>
                </div>

                <div class="q-mt-md" v-if="servicio.estadoEquipo === 'Entregado' && servicio.calificacion > 0">
                  <span class="text-grey-7 q-mr-sm">Calificación:</span>
                  <q-rating v-model="servicio.calificacion" size="1.5em" color="orange" readonly />
                </div>
                
                <div class="q-mt-md text-caption text-grey-8" v-if="servicio.observaciones">
                  <strong>Obs:</strong> {{ servicio.observaciones }}
                </div>
              </q-card-section>

              <q-card-actions align="right" class="bg-grey-1">
                <q-btn flat color="primary" icon="edit" label="Editar" @click="editServicio(servicio)" />
                <q-btn flat color="negative" icon="delete" label="Eliminar" @click="askDelete(servicio.id)" />
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <q-page-sticky position="bottom-right" :offset="[18, 18]">
          <q-btn fab icon="add" color="primary" @click="openModal" />
        </q-page-sticky>

        <q-dialog v-model="isModalOpen" persistent>
          <q-card style="width: 100%; max-width: 600px;">
            <q-form @submit="saveServicio">
              <q-toolbar class="bg-primary text-white">
                <q-toolbar-title>{{ isEditing ? 'Editar Servicio' : 'Nuevo Servicio' }}</q-toolbar-title>
                <q-btn flat round dense icon="close" v-close-popup />
              </q-toolbar>

              <q-card-section class="q-pa-md q-gutter-md">
                <div class="text-subtitle1 q-mt-none text-primary font-weight-bold">Datos del Cliente y Equipo</div>
                
                <q-input outlined v-model="form.cliente" label="Nombre del cliente *" :rules="[requiredRule]" />
                <q-input outlined v-model="form.equipo" label="Marca y modelo del equipo *" placeholder="Ej. Samsung A15" :rules="[requiredRule]" />
                
                <div class="text-subtitle1 q-mt-md text-primary font-weight-bold">Detalles de Reparación</div>

                <q-select outlined v-model="form.reparacion" :options="tiposReparacion" label="Tipo de reparación *" :rules="[requiredRule]" />
                <q-select outlined v-model="form.tecnico" :options="tecnicos" label="Técnico que atiende *" :rules="[requiredRule]" />
                <q-input outlined v-model="form.fechaRecepcion" type="datetime-local" label="Fecha y hora de recepción *" :rules="[requiredRule]" stack-label />
                
                <div class="text-subtitle1 q-mt-md text-primary font-weight-bold">Pago e Ingreso</div>
                
                <q-input outlined v-model="form.precio" type="number" label="Precio cobrado *" :rules="[priceRule]" prefix="$" />
                <q-select outlined v-model="form.metodoPago" :options="metodosPago" label="Método de pago *" :rules="[requiredRule]" />

                <q-select outlined v-model="form.estadoPago" :options="estadosPago" label="Estado del pago *" :rules="[requiredRule]" />
                <q-select outlined v-model="form.estadoEquipo" :options="estadosEquipo" label="Estado del equipo *" :rules="[requiredRule]" />
                
                <div v-show="form.estadoEquipo === 'Entregado'" class="q-mt-md">
                  <div class="text-subtitle2 q-mb-sm text-grey-8">Calificación del cliente</div>
                  <q-rating v-model="form.calificacion" size="2em" :max="5" color="orange" />
                </div>

                <q-input outlined v-model="form.observaciones" type="textarea" label="Observaciones (Opcional)" placeholder="Ej. pantalla rota en la esquina..." class="q-mt-md" />

              </q-card-section>

              <q-card-actions align="right" class="q-pa-md bg-white">
                <q-btn flat label="Cancelar" color="negative" v-close-popup />
                <q-btn type="submit" label="Guardar" color="primary" />
              </q-card-actions>
            </q-form>
          </q-card>
        </q-dialog>

        <q-dialog v-model="isDeleteModalOpen">
          <q-card>
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="red" text-color="white" />
              <span class="q-ml-sm">¿Estás seguro de que deseas eliminar este registro?</span>
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="Cancelar" color="primary" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" @click="deleteServicio" />
            </q-card-actions>
          </q-card>
        </q-dialog>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<style scoped>
</style>
