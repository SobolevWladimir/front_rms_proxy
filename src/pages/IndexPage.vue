<template>
  <q-page>
    <input type="file" ref="file" style="display: none" @change="onChange" />
    <q-dialog
      v-model="showInfoDialog"
      persistent
      maximized
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="">
        <q-bar>
          <q-space />

          <q-btn dense flat icon="close" v-close-popup>
            <q-tooltip class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section>
          <div class="text-h6">Запрос</div>
        </q-card-section>

        <q-card-section class="q-pt-none ">
          <textarea class="full-width" :value="modalData.clientRequest.Body"></textarea>
        </q-card-section>

        <q-card-section>
          <div class="text-h6">Ответ</div>
        </q-card-section>

        <q-card-section class="q-pt-none full-width row wrap justify-start items-start content-end">
          <textarea class="full-width" :value="modalData.clientResponse.Body"></textarea>
        </q-card-section>
      </q-card>
    </q-dialog>
    <div class="q-pa-sm">
      <q-btn label="Экспорт данных" @click="exportData"></q-btn>
      <q-btn label="Импорт данных" @click="$refs.file.click()"></q-btn>
    </div>
    <q-table flat bordered :rows="clientData" :columns="columns" row-key="name">
      <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <div>
            <q-btn outline no-caps icon="info" @click="showInfo(props.row)"></q-btn>
          </div>
        </q-td>
      </template>
      <template v-slot:body-cell-method="props">
        <q-td :props="props">
          <div>
            <q-badge :color="getMethodColor(props.val)" :label="props.value" />
          </div>
        </q-td>
      </template>
    </q-table>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'

const clientData = ref([])
const modalData = ref(null);
const showInfoDialog = ref(false);
const columns = [
  {
    name: 'action',
    label: '',
  },
  {
    name: 'method',
    label: 'Метод',
    field: (row) => row.clientRequest.Method,
    sortable: true,
  },

  {
    name: 'url',
    label: 'URL',
    align: 'left',
    field: (row) => row.clientRequest.URL,
    sortable: true,
  },
  {
    name: 'isErrorResponse',
    label: 'Ошибка',
    align: 'left',
    field: (row) => (row.isErrorResponse ? 'Да' : 'Нет'),
    sortable: true,
  },
  {
    name: 'isProxy',
    label: 'Перенаправлен?',
    align: 'left',
    field: (row) => (row.isProxy ? 'Да' : 'Нет'),
    sortable: true,
  },
  {
    name: 'proxyTo',
    label: 'Перенаправлен на',
    align: 'left',
    field: (row) => {
      if (row.isProxy) {
        if (row.proxyTo.replaceByFakeRms) {
          return row.proxyTo.fakeRms.url
        }
        return row.proxyTo.fakeRms.url
      }
      return row.mainRms.url
    },
    sortable: true,
  },
]
const socket = new WebSocket('ws://127.0.0.1:9090/ws')
function getMethodColor(val) {
  if (val == 'GET') {
    return 'green'
  }
  if (val == 'POST') {
    return 'yellod'
  }
  if (val == 'PUT') {
    return 'blue'
  }
  if (val == 'DELETE') {
    return 'red'
  }
  return 'orange'
}
const isConnected = ref(false)

socket.onopen = () => {
  console.log('WebSocket connection established')
  isConnected.value = true
}

socket.onmessage = (event) => {
  const data = JSON.parse(event.data)
  clientData.value = clientData.value.concat(data)

  console.log('Real-time update:', data)
}

socket.onerror = (error) => {
  console.error('WebSocket error:', error)
}

socket.onclose = () => {
  console.log('WebSocket connection closed')
  isConnected.value = false
}

function exportData() {
  const filename = 'dump.json'
  const text = JSON.stringify(clientData.value)

  let element = document.createElement('a')
  element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text))
  element.setAttribute('download', filename)
  element.style.display = 'none'
  document.body.appendChild(element)
  element.click()
  document.body.removeChild(element)
}
function onChange(e) {
  let files = e.target.files
  if (files.length == 0) {
    return
  }
  let file = files[0]
  let reader = new FileReader()

  reader.readAsText(file)

  reader.onload = function () {
    let data = JSON.parse(reader.result)
    clientData.value = data
  }

  reader.onerror = function () {
    console.log(reader.error)
  }
}
function showInfo(data){
  modalData.value = data
  console.log(modalData)
  showInfoDialog.value = true;
}

//
</script>
