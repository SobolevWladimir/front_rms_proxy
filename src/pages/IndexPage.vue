<template>
  <q-page>
    <q-table flat bordered title="Запросы" :rows="clientData" :columns="columns" row-key="name" >
       <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <div>
            <q-btn outline  no-caps  icon="info"></q-btn>

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
function getMethodColor(val){
  if(val == "GET"){
    return 'green'
  }
  if(val == "POST"){
    return 'yellod'
  }
  if(val == "PUT"){
    return 'blue'
  }
  if(val == "DELETE"){
    return 'red'
  }
  return 'orange'
}

socket.onopen = () => {
  console.log('WebSocket connection established')
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
}

//
</script>
