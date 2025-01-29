<template>
  <q-card class="">
    <q-bar>
      <q-space />

      <q-btn dense flat icon="close" v-close-popup>
        <q-tooltip class="bg-white text-primary">Close</q-tooltip>
      </q-btn>
    </q-bar>
    <q-tabs v-model="tab" dense class="bg-grey-2 text-grey-7" active-color="primary" indicator-color="purple"
      align="justify">
      <q-tab name="request" label="Запрос" />
      <q-tab name="response" label="Ответ" />
      <q-tab name="proxy" label="Прокси информация" />
      <q-tab name="main" label="Дерево" />
    </q-tabs>

    <q-tab-panels v-model="tab" animated>
      <q-tab-panel name="request">
        <q-card-section>
          <div class="text-h6">Заголовки</div>
        </q-card-section>
        <table>
          <tr v-for="(val, key) in rdata.clientRequest.Header" :key="key">
            <td>{{ key }}</td>
            <td>{{ val }}</td>
          </tr>
        </table>

        <q-card-section>
          <div class="text-h6">Тело</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <q-input :modelValue="rdata.clientRequest.Body" class="full-width" filled type="textarea" />
        </q-card-section>
      </q-tab-panel>

      <q-tab-panel name="response">
        <q-card-section>
          <div class="text-h6">Заголовки</div>
        </q-card-section>
        <table>
          <tr v-for="(val, key) in rdata.clientResponse.Header" :key="key">
            <td>{{ key }}</td>
            <td>{{ val }}</td>
          </tr>
        </table>

        <q-card-section>
          <div class="text-h6">Тело</div>
        </q-card-section>
        <q-card-section class="q-pt-none full-width row wrap justify-start items-start content-end">
          <q-input :modelValue="rdata.clientResponse.Body" class="full-width" filled type="textarea" />
        </q-card-section>
      </q-tab-panel>

      <q-tab-panel name="proxy">
        <div v-if="rdata.isProxy">
          <div>Спроксированно на</div>
          {{ rdata.proxyTo }}
        </div>
        <div v-else>
          <div>На основную рмс</div>
          {{ rdata.mainRms }}
        </div>
      </q-tab-panel>

      <q-tab-panel name="main">
        <q-card-section>
          <div class="text-h6">Запрос</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <XmlViewer v-if="isXml(rdata.clientRequest.Header)" :xml="rdata.clientRequest.Body" />
          <div v-else>{{ rdata.clientRequest.Body }}</div>
        </q-card-section>

        <q-card-section>
          <div class="text-h6">Ответ</div>
        </q-card-section>

        <q-card-section class="q-pt-none full-width row wrap justify-start items-start content-end">
          <q-scroll-area class="full-width rounded-borders bg-grey-1" style="height: 400px">
            <XmlViewer v-if="isXml(rdata.clientResponse.Header)" :xml="rdata.clientResponse.Body" />
            <div v-else>{{ rdata.clientResponse.Body }}</div>
          </q-scroll-area>
        </q-card-section>
      </q-tab-panel>
    </q-tab-panels>
  </q-card>
</template>
<script setup>
import { ref } from 'vue'
import XmlViewer from 'vue3-xml-viewer'
defineProps(['rdata'])
const tab = ref('request')
function isXml(header) {
  if ('Content-Type' in header) {
    let data = JSON.parse(JSON.stringify(header['Content-Type']))
    console.log('test', data[0])
    return data[0] == 'application/xml'
  }

  return false
}
</script>
