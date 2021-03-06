<template>
  <div class="welcome">
    <header>
      <h1>
        <img src="../assets/logo.svg" alt="Grapefruit" width="160" id="logo" />
      </h1>
      <h2 class="subtitle">Runtime app instruments for iOS</h2>
      <!-- <b-button @click="connect">Add Remote TCP</b-button> -->
      <aside class="menu">
        <p class="menu-label">Frida version: {{ version }}</p>

        <hr />

        <p class="menu-label">
          Devices
          <loading v-if="loading" size="24" class="is-pulled-right"></loading>
        </p>

        <ul class="menu-list device-list">
          <li v-for="dev in devices" :key="dev.id">
            <router-link :to="{ name: 'Apps', params: { device: dev.id } }">
              <!-- <icon :icon="dev.icon" :width="24" :height="24"></icon> -->
              <b-icon icon="cellphone" size="is-small" />
              {{ dev.name }}
              <b-button
                v-if="dev.removable"
                class="remove"
                icon-right="delete"
                type="is-danger"
                @click.stop.prevent="remove(dev.id)"
              ></b-button>
            </router-link>
          </li>

          <li v-if="!loading && !devices.length">
            <b-icon icon="lan-disconnect" type="is-danger" />No device found
          </li>

          <li class="add-remote">
            <b-button expanded icon-left="plus-circle-outline">Connect Remote ...</b-button>
          </li>
        </ul>

        <p class="menu-label">Support</p>
        <ul class="menu-list">
          <li>
            <a target="_blank" href="https://github.com/chichou/grapefruit">
              <b-icon type="is-small" icon="github" />&nbsp;GitHub
            </a>
          </li>
          <li>
            <a target="_blank" href="https://discordapp.com/invite/pwutZNx">
              <b-icon type="is-small" icon="discord" />&nbsp;Discord
            </a>
          </li>
        </ul>
      </aside>
    </header>

    <router-view class="main"></router-view>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import Axios from 'axios'
import * as io from 'socket.io-client'

import Icon from '../components/Icon.vue'
import Loading from '../components/Loading.vue'

@Component({
  components: {
    Icon,
    Loading
  }
})
export default class Welcome extends Vue {
  version = 'N/A'
  devices = []
  loading = false

  mounted() {
    const socket = io.connect('/devices', { transports: ['websocket'] })
    socket.on('deviceChanged', this.refresh)
    this.refresh()
  }

  refresh() {
    this.loading = true
    Axios.get('/devices')
      .then(({ data }) => {
        this.version = data.version
        this.devices = data.list
      })
      .catch(e => {
        this.$buefy.dialog.alert({
          title: 'Failed to load devices',
          message: e.response.data,
          type: 'is-danger',
          hasIcon: true,
          icon: 'close-circle',
          ariaRole: 'alertdialog',
          ariaModal: true
        })
      })
      .finally(() => {
        this.loading = false
      })
  }
}
</script>

<style lang="scss" scoped>
.welcome {
  display: flex;
  flex-direction: row;

  header {
    width: 320px;
    padding: 40px;
    min-height: 100vh;
    height: 100%;
    background: rgba(0, 0, 0, 0.2);
    position: sticky;
    top: 0;
  }

  .main {
    flex: 1;
  }
}

h1 {
  margin: 0;
  font-weight: 100;
}

.device-list {
  canvas {
    margin-right: 4px;
  }

  .add-remote {
    margin-top: 10px;
  }

  a {
    display: flex;
    align-items: center;
  }

  .icon {
    margin-right: 4px;
  }

  button {
    margin-left: auto;
  }
}
</style>
