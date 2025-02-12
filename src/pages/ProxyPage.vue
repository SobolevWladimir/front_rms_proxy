<template>
  <q-page padding>
    <q-dialog v-model="showEditModal">
      <q-card style="width: 600px">
        <q-card-section>
          <div v-if="isEdit" class="text-h6">Редактировать</div>
          <div v-else class="text-h6">Создать</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-subtitle1">Блок Условий</div>
          <q-form @submit="onSubmit" class="q-gutter-md">
            <q-input
              filled
              v-model="editForm.path"
              label="Путь"
              hint=""
              lazy-rules
              :rules="[(val) => (val && val.length > 0) || 'ЭЭЭЭЭ']"
            />

            <div class="text-subtitle2">query параметры</div>
            <table>
              <tbody>
                <tr v-for="item in editForm.query_keys" :key="item.key">
                  <td>
                    <q-input filled v-model="item.key" label="Ключ" />
                  </td>
                  <td>
                    <q-input filled v-model="item.value" label="Значение" />
                  </td>
                </tr>
              </tbody>
            </table>
            <q-btn
              size="sm"
              label="Добавить"
              @click="editForm.query_keys.push({ key: '', value: '' })"
            />

            <q-input
              filled
              v-model="editForm.content"
              type="textarea"
              label="Тело запроса"
              hint=""
              lazy-rules
            />

            <q-checkbox v-model="editForm.is_content_contains" label="Частичное соответствие" />

            <q-separator />
            <div class="text-subtitle1">Блок результата</div>
            <q-checkbox v-model="editForm.replaceByFakeRms" label="Перенаправить на другую рмс" />

            <q-input
              v-if="editForm.replaceByFakeRms == false"
              filled
              v-model="editForm.fakeContent"
              type="textarea"
              label="Вернуть эти данные"
              hint=""
              lazy-rules
            />

            <div>{{ editForm.fakeRmsId }}</div>
            <q-select
              v-if="editForm.replaceByFakeRms == true"
              v-model="editForm.fakeRmsId"
              :options="rmsList.list"
              option-value="id"
              option-label="name"
              label="Перенаправить на рмс"
              emit-value
              map-options
            />

            <q-input
              v-if="editForm.replaceByFakeRms == true"
              filled
              v-model="editForm.pathTo"
              label="Подменить путь на"
              hint=""
              lazy-rules
              @focus="
                () => {
                  if (editForm.pathTo.length == 0) {
                    editForm.pathTo = editForm.path
                  }
                }
              "
            />

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
          <q-btn
            color="blue"
            icon="edit"
            @click="editItem(props.row, props.rowIndex)"
            size="sm"
            no-caps
          ></q-btn>
          <q-btn
            color="red"
            icon="delete"
            @click="deleteItem(props.rowIndex)"
            size="sm"
            no-caps
          ></q-btn>
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
  content: '',
  fakeRmsId: null,
  fakeContent: 'dfsdfsd',
  is_content_contains: false,
  query_keys: [],
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
    field: (row) => (row.fakeContent.length > 50 ? row.fakeContent.slice(0, 50) : row.fakeContent),
    sortable: true,
    // style: "max-width: 150px"
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
  editForm.value.is_content_contains = false
  editForm.value.query_keys = []
  editForm.value.content = ''

  isEdit.value = false
  showEditModal.value = true
}
function editItem(item, index) {
  let queryKeys = []
  for (let key in item.query_keys) {
    queryKeys.push({
      key,
      value: item.query_keys[key],
    })
  }

  editForm.value.path = item.path
  editForm.value.pathTo = item.pathTo
  editForm.value.replaceByFakeRms = item.replaceByFakeRms
  editForm.value.fakeRmsId = item.fakeRmsId
  editForm.value.fakeContent = item.fakeContent
  editForm.value.content = item.content
  editForm.value.is_content_contains = item.is_content_contains
  editForm.value.query_keys = queryKeys
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

  let qur = {}
  for (let i = 0; i < editForm.value.query_keys.length; i++) {
    const item = editForm.value.query_keys[i]
    if(item.key.length>0){
    qur[item.key] = item.value

    }
  }
  if (isEdit.value == false) {
    let data = JSON.parse(JSON.stringify(editForm.value))
    data.query_keys = qur
    pageData.value.push(data)
    saveDataOnServer()
    return
  }
  let item = pageData.value[editIndex]


  item.path = editForm.value.path
  item.replaceByFakeRms = editForm.value.replaceByFakeRms
  item.fakeRmsId = editForm.value.fakeRmsId
  item.pathTo = editForm.value.pathTo
  item.fakeContent = editForm.value.fakeContent
  item.content = editForm.value.content
  item.is_content_contains = editForm.value.is_content_contains
  item.query_keys = qur
  pageData.value[editIndex] = item
  saveDataOnServer()
}
function saveDataOnServer() {
  axios.post('/setting/proxy', pageData.value)
}

//
</script>
