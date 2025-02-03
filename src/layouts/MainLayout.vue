<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn stretch flat label="Логи"  to="/" />
        <q-btn stretch flat label="Настройки прокси" to="/proxy" />
        <q-btn stretch flat label="RMS/Chain" to="/rms" />
        <q-space />
        <div>
          <q-toolbar-title>RMSProxy</q-toolbar-title>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup></script>
