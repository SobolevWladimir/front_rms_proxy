<template>
  <q-page padding>
    <q-dialog v-model="showEditModal">
      <q-card style="width: 300px">
        <q-card-section>
          <div v-if="isEdit" class="text-h6">Редактировать</div>
          <div v-else class="text-h6">Создать</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-form @submit="onSubmit" class="q-gutter-md">
            <q-input filled v-model="editForm.name" label="Имя" hint="" lazy-rules
              :rules="[(val) => (val && val.length > 0) || 'Обзови уже как нибудь']" />

            <q-input filled v-model="editForm.url" label="Url" hint="" lazy-rules
              :rules="[(val) => (val && val.length > 0) || 'У тебя все дома?']" />

            <q-input filled v-model="editForm.login" label="Логин" hint="" lazy-rules />
            <q-input filled v-model="editForm.password" label="Пароль" hint="" lazy-rules />
            <q-checkbox v-model="editForm.needPassEncrupt" label="Зашифровать пароль в  sha1" />

            <q-input filled v-model="editForm.listenPort" label="Порт" hint="Если не знаешь для чего, оставь пустым"
              lazy-rules />

            <div>
              <q-btn label="Сохранить" type="submit" color="primary" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-table flat bordered :rows="pageData.list" :columns="columns" row-key="id" :filter="filter">
      <template v-slot:top>
        <q-btn color="primary" :disable="loading" label="Добавить" @click="addRow" />
        <q-space />
        <q-input borderless dense debounce="300" color="primary" v-model="filter">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>
      <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <q-btn color="blue" icon="edit" @click="editItem(props.row)" size="sm" no-caps></q-btn>
          <q-btn color="red" icon="delete" @click="deleteItem(props.rowIndex)" size="sm" no-caps></q-btn>
        </q-td>
      </template>
    </q-table>
  </q-page>
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'
import { UUID } from 'uuidjs'

const loading = ref(false)
const pageData = ref({ list: [] })
const filter = ref('')
const showEditModal = ref(false)
const isEdit = ref(false)
const editForm = ref({
  id: 'rmsChain',
  name: 'Демо Chain',
  url: 'http://192.168.0.155:9080',
  login: 'adm',
  password: '123',
  needPassEncrupt: true,
  listenPort: '',
})

const columns = [
  {
    name: 'name',
    label: 'Имя',
    field: (row) => row.name,
    sortable: true,
  },
  {
    name: 'url',
    label: 'URL',
    field: (row) => row.url,
    sortable: true,
  },
  {
    name: 'login',
    label: 'login',
    field: (row) => row.login,
    sortable: true,
  },

  {
    name: 'password',
    label: 'Пароль',
    field: (row) => row.password,
    sortable: true,
  },
  {
    name: 'needPassEncrupt',
    label: 'Шифровать пароль в sha1',
    field: (row) => (row.needPassEncrupt ? 'Да' : 'Нет'),
    sortable: true,
  },
  {
    name: 'listenPort',
    label: 'Слушать порт',
    field: (row) => row.listenPort,
    sortable: true,
  },
  {
    name: 'id',
    label: 'id',
    field: (row) => row.id,
    sortable: true,
  },
  {
    name: 'action',
    label: '',
  },
]

async function loadRmsList() {
  loading.value = true
  let data = await axios.get('/setting/rms')

  pageData.value.list = data.data.list
  loading.value = false
}
function addRow() {
  editForm.value.id = UUID.generate()
  editForm.value.name = ''
  editForm.value.url = ''
  editForm.value.login = ''
  editForm.value.needPassEncrupt = true
  editForm.value.listenPort = ''
  isEdit.value = false
  showEditModal.value = true
}
function editItem(item) {
  editForm.value.id = item.id
  editForm.value.name = item.name
  editForm.value.url = item.url
  editForm.value.login = item.login
  editForm.value.password = item.password
  editForm.value.needPassEncrupt = item.needPassEncrupt
  editForm.value.listenPort = item.listenPort
  isEdit.value = true
  showEditModal.value = true
}
function deleteItem(index) {
  pageData.value.list.splice(index, 1)
  saveDataOnServer()
}

function onSubmit() {
  showEditModal.value = false
  if (isEdit.value == false) {
    pageData.value.list.push(JSON.parse(JSON.stringify(editForm.value)))
    saveDataOnServer()
    return
  }
  for (let i = 0; i < pageData.value.list.length; i++) {
    let item = pageData.value.list[i]

    if (item.id !== editForm.value.id) {
      continue
    }

    item.id = editForm.value.id
    item.name = editForm.value.name
    item.url = editForm.value.url
    item.login = editForm.value.login
    item.password = editForm.value.password
    item.needPassEncrupt = editForm.value.needPassEncrupt
    item.listenPort = editForm.value.listenPort
    pageData.value.list[i] = item
  }
  saveDataOnServer()
}
function saveDataOnServer() {
  axios.post('/setting/rms', pageData.value)
}
loadRmsList()

//
</script>
