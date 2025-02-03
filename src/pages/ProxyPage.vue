<template>
  <q-page padding>
    <q-dialog v-model="showEditModal">
      <q-card style="width: 600px">
        <q-card-section>
          <div v-if="isEdit" class="text-h6">Редактировать</div>
          <div v-else class="text-h6">Создать</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-form @submit="onSubmit" class="q-gutter-md">
            <q-input filled v-model="editForm.path" label="Путь" hint="" lazy-rules
              :rules="[(val) => (val && val.length > 0) || 'ЭЭЭЭЭ']" />

            <q-checkbox v-model="editForm.replaceByFakeRms" label="Перенаправить на другую рмс" />

            <q-input v-if="editForm.replaceByFakeRms == false" filled v-model="editForm.fakeContent" type="textarea"
              label="Вернуть эти данные" hint="" lazy-rules />

            <div>{{ editForm.fakeRmsId }}</div>
            <q-select v-if="editForm.replaceByFakeRms == true" v-model="editForm.fakeRmsId" :options="rmsList.list"
              option-value="id" option-label="name" label="Перенаправить на рмс" emit-value map-options />

            <q-input v-if="editForm.replaceByFakeRms == true" filled v-model="editForm.pathTo" label="Подменить путь на"
              hint="" lazy-rules @focus="() => {
                  if (editForm.pathTo.length == 0) {
                    editForm.pathTo = editForm.path
                  }
                }
                " />

            <div>
              <q-btn label="Сохранить" type="submit" color="primary" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-table flat bordered :rows="pageData" :columns="columns" row-key="id" :filter="filter">
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
          <q-btn color="blue" icon="edit" @click="editItem(props.row, props.rowIndex)" size="sm" no-caps></q-btn>
          <q-btn color="red" icon="delete" @click="deleteItem(props.rowIndex)" size="sm" no-caps></q-btn>
        </q-td>
      </template>
    </q-table>
  </q-page>
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'

const loading = ref(false)
const pageData = ref([])
const rmsList = ref({ list: [] })
const filter = ref('')
const showEditModal = ref(false)
const isEdit = ref(false)
const editForm = ref({
  path: '/method/test',
  pathTo: '',
  replaceByFakeRms: false,
  fakeRmsId: null,
  fakeContent: 'dfsdfsd',
})
let editIndex = 0

const columns = [
  {
    name: 'path',
    label: 'Путь',
    field: (row) => row.path,
    sortable: true,
  },
  {
    name: 'replaceByFakeRms',
    label: 'Перенаправлять на другую рмс',
    field: (row) => (row.replaceByFakeRms ? 'Да' : 'Нет'),
    sortable: true,
  },
  {
    name: 'fakeContent',
    label: 'Подменить на контент',
    field: (row) => row.fakeContent,
    sortable: true,
  },
  {
    name: 'fakeRmsId',
    label: 'Перенаправить на РМС',
    field: (row) => row.fakeRmsId,
    sortable: true,
  },
  {
    name: 'pathTo',
    label: 'Заменить путь на',
    field: (row) => row.pathTo,
    sortable: true,
  },
  {
    name: 'action',
    label: '',
  },
]

async function loadRmsList() {
  let data = await axios.get('/setting/rms')

  rmsList.value.list = data.data.list
}

async function loadProxyList() {
  let data = await axios.get('/setting/proxy')

  pageData.value = data.data
}
async function loadData() {
  loading.value = true
  await loadRmsList()
  await loadProxyList()
  loading.value = false
}

loadData()

function addRow() {
  editForm.value.path = ''
  editForm.value.pathTo = ''
  editForm.value.replaceByFakeRms = false
  editForm.value.fakeRmsId = null
  editForm.value.fakeContent = ''
  isEdit.value = false
  showEditModal.value = true
}
function editItem(item, index) {
  editForm.value.path = item.path
  editForm.value.pathTo = item.pathTo
  editForm.value.replaceByFakeRms = item.replaceByFakeRms
  editForm.value.fakeRmsId = item.fakeRmsId
  editForm.value.fakeContent = item.fakeContent
  isEdit.value = true
  showEditModal.value = true
  editIndex = index
}
function deleteItem(index) {
  pageData.value.list.splice(index, 1)
  saveDataOnServer()
}

function onSubmit() {
  showEditModal.value = false
  if (isEdit.value == false) {
    pageData.value.push(JSON.parse(JSON.stringify(editForm.value)))
    saveDataOnServer()
    return
  }
  let item = pageData.value[editIndex]

  item.path = editForm.value.path
  item.replaceByFakeRms = editForm.value.replaceByFakeRms
  item.fakeRmsId = editForm.value.fakeRmsId
  item.pathTo = editForm.value.pathTo
  item.fakeContent = editForm.value.fakeContent
  pageData.value[editIndex] = item
  saveDataOnServer()
}
function saveDataOnServer() {
  axios.post('/setting/proxy', pageData.value)
}

//
</script>
