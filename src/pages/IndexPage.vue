<template>
  <q-page>
    <input type="file" ref="file" style="display: none" @change="onChange" />
    <q-dialog
      maximized
      v-model="showInfoDialog"
      persistent
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <RequestInfo :rdata="modalData" />
    </q-dialog>
    <q-table
      flat
      bordered
      :rows="clientData"
      :columns="columns"
      :pagination="tablePagination"
      :filter="filter"
      row-key="url"
    >
      <template v-slot:top>
        <q-btn label="Экспорт данных" @click="exportData"></q-btn>
        <q-btn label="Импорт данных" @click="$refs.file.click()"></q-btn>
        <q-btn label="Очистить" icon="delete_forever" @click="clientData = []"></q-btn>
        <q-space />
        <q-input borderless dense debounce="300" color="primary" v-model="filter">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>
      <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <div>
            <q-btn color="blue" size="sm" no-caps icon="info" @click="showInfo(props.row)"></q-btn>
          </div>
        </q-td>
      </template>
      <template v-slot:body-cell-method="props">
        <q-td :props="props">
          <div>
            <q-badge :color="getMethodColor(props.value)" :label="props.value" />
          </div>
        </q-td>
      </template>
      <template v-slot:body-cell-status="props">
        <q-td :props="props">
          <div>
            <q-badge :color="getStatusColor(props.value)" :label="props.value" />
          </div>
        </q-td>
      </template>
    </q-table>
  </q-page>
</template>

<script setup>
import { ref, isProxy, toRaw } from 'vue'
import RequestInfo from '../components/RequestInfo.vue'

const clientData = ref([])
const modalData = ref(null)
const showInfoDialog = ref(false)
const filter = ref('')
const tablePagination = ref({
  sortBy: null,
  descending: false,
  page: 1,
  rowsPerPage: 1000,
  // rowsNumber: xx if getting data from a server
})
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
    name: 'status',
    label: 'Статус',
    align: 'left',
    field: (row) => row.clientResponse.Status,
    sortable: true,
  },
  {
    name: 'user-agent',
    label: 'AuthType',
    align: 'left',
    // field: (row) => row.clientRequest.Header['User-Agent'],
    field: (row) => {
      if (isProxy(row.clientRequest.Header)) {
        const headers = toRaw(row.clientRequest.Header)
        const result = headers['X-Resto-Authtype']
        if (result) {
          return result[0]
        }
      }
      return ''
    },
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
      function getName(rms) {
        return `${rms.name}(${rms.url})`
      }
      if (row.isProxy) {
        if (row.proxyTo.replaceByFakeRms) {
          return getName(row.proxyTo.fakeRms)
        }
        return 'Вернули фейк данные'
      }
      return getName(row.mainRms)
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
    return 'yellow'
  }
  if (val == 'PUT') {
    return 'blue'
  }
  if (val == 'DELETE') {
    return 'red'
  }
  return 'orange'
}

function getStatusColor(val) {
  if (val == 200 || val == 201) {
    return 'green'
  }
  return 'red'
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
    clientData.value = clientData.value.concat(data)
  }

  reader.onerror = function () {
    console.log(reader.error)
  }
}
function showInfo(data) {
  modalData.value = data
  console.log(modalData)
  showInfoDialog.value = true
}

//
</script>
