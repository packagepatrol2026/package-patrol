<template>
  <ion-page>
    <ion-content>
      <div class="connect-layout ion-padding">
      <div
        class="logo-wrap"
        :class="{ 'logo-wrap--after-unknown-banner': lock.status === 'UNKNOWN' }"
      >
        <img
          class="app-logo"
          src="/logo.png"
          width="400"
          height="400"
          alt="Package Patrol"
        />
      </div>

      <h2>{{ locale.t('connect.title') }}</h2>
      <p class="muted">{{ locale.t('connect.notConnected') }}</p>

      <ion-note class="platform-note" color="medium">
        <p v-if="isIos">{{ locale.t('connect.iosHint') }}</p>
        <p v-else-if="isWeb">{{ locale.t('connect.webHint') }}</p>
        <p v-else>{{ locale.t('connect.nativeHint') }}</p>
      </ion-note>

      <p v-if="connection.isOnline" class="status-ok">
        {{ locale.t('connect.connectedTo', { name: connection.deviceName ?? '—' }) }}
      </p>
      <p v-if="connection.lastError" class="status-err">
        {{ locale.t('connect.error', { msg: connection.lastError }) }}
      </p>

      <ion-button expand="block" color="primary" class="ion-margin-top pair-btn" @click="onPair" :disabled="busy">
        {{ locale.t('connect.pair') }}
      </ion-button>
      <ion-button
        expand="block"
        fill="outline"
        color="medium"
        :disabled="!connection.isOnline || busy"
        @click="onDisconnect"
      >
        {{ locale.t('connect.disconnect') }}
      </ion-button>

      <footer class="connect-footer">
        <p class="footer-contact">
          <span class="footer-label">{{ locale.t('connect.contactUs') }}</span>
          <a class="footer-link" :href="`mailto:${CONTACT_EMAIL}`">{{ CONTACT_EMAIL }}</a>
        </p>
        <p class="footer-copyright">{{ locale.t('connect.copyright') }}</p>
      </footer>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonButton, IonContent, IonNote, IonPage } from '@ionic/vue';
import { Capacitor } from '@capacitor/core';
import { computed, ref } from 'vue';
import { useBluetooth } from '@/composables/useBluetooth';
import { useConnectionStore } from '@/stores/connection';
import { useLockStore } from '@/stores/lock';
import { useLocaleStore } from '@/stores/locale';

const locale = useLocaleStore();
const connection = useConnectionStore();
const lock = useLockStore();
const { pairAndConnect, disconnect } = useBluetooth();

const CONTACT_EMAIL = 'package.patrol.2026@gmail.com';

const busy = ref(false);

const isIos = computed(() => Capacitor.getPlatform() === 'ios');
const isWeb = computed(() => Capacitor.getPlatform() === 'web');

async function onPair() {
  busy.value = true;
  connection.clearError();
  try {
    await pairAndConnect();
  } catch (e) {
    const msg = e instanceof Error ? e.message : String(e);
    connection.setError(msg);
  } finally {
    busy.value = false;
  }
}

async function onDisconnect() {
  busy.value = true;
  try {
    await disconnect();
  } finally {
    busy.value = false;
  }
}
</script>

<style scoped>
.connect-layout {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
}

.logo-wrap {
  display: flex;
  justify-content: center;
  margin: 0 0 1.25rem;
  padding: 1rem 0.75rem;
  background: #aca299;
  border-radius: 12px;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.25);
}

/* Breathing room below lock banner (“unknown — connect and sync”) on Connect tab */
.logo-wrap--after-unknown-banner {
  margin-top: 1.25rem;
}

.app-logo {
  width: min(100%, 400px);
  height: auto;
  max-height: min(42vh, 300px);
  object-fit: contain;
  display: block;
}

.pair-btn {
  font-weight: 600;
}

.muted {
  opacity: 0.85;
  margin-top: 0;
}
.platform-note {
  display: block;
  margin: 1rem 0;
  font-size: 0.9rem;
  line-height: 1.4;
}
.status-ok {
  color: var(--ion-color-success-shade);
  font-weight: 600;
}
.status-err {
  color: var(--ion-color-danger-shade);
  font-size: 0.9rem;
}

.connect-footer {
  margin-top: auto;
  padding-top: 2.5rem;
  padding-bottom: 0.5rem;
  text-align: center;
  font-size: 0.875rem;
  color: var(--ion-color-medium);
}

.footer-contact {
  margin: 0 0 0.5rem;
  line-height: 1.5;
}

.footer-label {
  display: block;
  margin-bottom: 0.25rem;
  font-weight: 600;
  color: var(--ion-text-color);
  opacity: 0.85;
}

.footer-link {
  color: var(--ion-color-primary);
  text-decoration: none;
  word-break: break-all;
}

.footer-link:active {
  opacity: 0.8;
}

.footer-copyright {
  margin: 0;
  font-size: 0.8125rem;
  opacity: 0.9;
}
</style>
