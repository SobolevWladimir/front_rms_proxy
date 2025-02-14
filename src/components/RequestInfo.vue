<template>
  <q-card class="">
    <q-bar>
      <q-space />

      <q-btn dense flat icon="close" v-close-popup>
        <q-tooltip class="bg-white text-primary">Close</q-tooltip>
      </q-btn>
    </q-bar>
    <q-tabs
      v-model="tab"
      dense
      class="bg-grey-2 text-grey-7"
      active-color="primary"
      indicator-color="purple"
      align="justify"
    >
      <q-tab name="request" label="Запрос" />
      <q-tab name="response" label="Ответ" />
      <q-tab name="proxy" label="Прокси информация" />
      <q-tab name="main" label="Дерево" />
    </q-tabs>

    <q-tab-panels v-model="tab" animated>
      <q-tab-panel name="request">
        <q-card-section>
          <table>
            <tbody>
              <tr>
                <td>Метод:</td>
                <td>
                  {{ rdata.clientRequest.Method }}
                </td>
              </tr>
              <tr>
                <td>URL:</td>
                <td>
                  {{ rdata.clientRequest.URL }}
                </td>
              </tr>
            </tbody>
          </table>
        </q-card-section>
        <q-card-section>
          <div class="text-h6">Заголовки</div>
        </q-card-section>
        <q-card-section>
          <table>
            <tbody>
              <tr v-for="(val, key) in rdata.clientRequest.Header" :key="key">
                <td>{{ key }}</td>
                <td>{{ val }}</td>
              </tr>
            </tbody>
          </table>
        </q-card-section>

        <q-card-section>
          <div class="text-h6">Тело</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <q-input
            :modelValue="rdata.clientRequest.Body"
            class="full-width"
            filled
            type="textarea"
          />
        </q-card-section>
      </q-tab-panel>

      <q-tab-panel name="response">
        <q-card-section>
          <div class="text-h6">Заголовки</div>
        </q-card-section>
        <q-card-section>
          <table>
            <tr v-for="(val, key) in rdata.clientResponse.Header" :key="key">
              <td>{{ key }}</td>
              <td>{{ val }}</td>
            </tr>
          </table>
        </q-card-section>

        <q-card-section>
          <div class="text-h6">Тело</div>
        </q-card-section>
        <q-card-section class="q-pt-none full-width row wrap justify-start items-start content-end">
          <q-input
            :modelValue="rdata.clientResponse.Body"
            class="full-width"
            filled
            type="textarea"
          />
        </q-card-section>
      </q-tab-panel>

      <q-tab-panel name="proxy">
        <q-card-section>
          <table>
            <tbody>
              <tr>
                <td>Был подменен:</td>
                <td>
                  {{ rdata.isProxy ? 'Да' : 'Нет' }}
                </td>
              </tr>
              <tr v-if="rdata.isProxy">
                <td>Перенаправлен на другую рмс:</td>
                <td>
                  {{ rdata.proxyTo.replaceByFakeRms ? 'Да' : 'Нет' }}
                </td>
              </tr>
              <tr v-if="rdata.isProxy && rdata.proxyTo.replaceByFakeRms">
                <td>Перенаправлен на</td>
                <td>
                  {{ rdata.proxyTo.fakeRms.name }}
                </td>
              </tr>

              <template v-if="rdata.isProxy && rdata.proxyTo.replaceByFakeRms">
                <tr>
                  <td>Метод:</td>
                  <td>
                    {{ rdata.clientProxyRequest.Method }}
                  </td>
                </tr>
                <tr>
                  <td>URL:</td>
                  <td>
                    {{ rdata.clientProxyRequest.URL }}
                  </td>
                </tr>
              </template>
              <tr v-if="rdata.isProxy && !rdata.proxyTo.replaceByFakeRms">
                <td>Вернули контент:</td>
                <td>Да</td>
              </tr>
              <tr v-if="!rdata.isProxy">
                <td>Запрос отправлен на :</td>
                <td>
                  {{ rdata.mainRms.name }}
                </td>
              </tr>
            </tbody>
          </table>
        </q-card-section>
      </q-tab-panel>

      <q-tab-panel name="main" class="row full-width">
        <div class="col">
          <q-card-section>
            <div class="text-h6">Запрос</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <XmlViewer v-if="isXml(rdata.clientRequest)" :xml="rdata.clientRequest.Body" />
            <div v-else>{{ rdata.clientRequest.Body }}</div>
          </q-card-section>
        </div>
        <div class="col">
          <q-card-section>
            <div class="text-h6">Ответ</div>
          </q-card-section>

          <q-card-section
            class="q-pt-none full-width row wrap justify-start items-start content-end"
          >
            <q-scroll-area class="full-width rounded-borders bg-grey-1" style="height: 800px">
              <XmlViewer v-if="isXml(rdata.clientResponse)" :xml="rdata.clientResponse.Body" />
              <div v-else>{{ rdata.clientResponse.Body }}</div>
            </q-scroll-area>
          </q-card-section>
        </div>
      </q-tab-panel>
    </q-tab-panels>
  </q-card>
</template>
<script setup>
import { ref } from 'vue'
import XmlViewer from 'vue3-xml-viewer'
defineProps(['rdata'])
const tab = ref('request')
function isXml(clientData) {
  if ('Content-Type' in clientData.Header) {
    let data = JSON.parse(JSON.stringify(clientData.Header['Content-Type']))
    if (data[0] == 'application/xml') {
      return true
    }
  }
  let body = clientData.Body
  if (body.length > 10) {
    body = clientData.Body.substring(0, 10)
    if (body.includes('<?xml')) {
      return true
    }
  }

  return false
}
</script>
