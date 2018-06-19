<template>
  <span v-if="!loading">
    <form @submit.prevent="sendDone">
      <p class="control">
        <label class="checkbox">
          <input v-model="client_mode" type="radio" :value="true"/> Client Mode
        </label>
        <br/>
        <label class="checkbox">
          <input v-model="client_mode" type="radio" :value="false"/> AccessPoint Mode
        </label>

      </p>

      <div v-if="!client_mode">

        <label class="label" for="wifi_ssid_ap">Wi-Fi SSID</label>
        <p class="control">
          <input v-model.trim="wifiSsidAp" class="input" type="text" id="wifi_ssid_ap" placeholder="Wi-Fi SSID"
                 required/>
          <span class="help">Required.</span>
        </p>

        <label class="label" for="wifi_password_ap">Wi-Fi Password</label>
        <p class="control is-grouped">
          <!-- v-model does not support dynamic :type -->
          <input v-if="passwordClearTextAp" type="text" class="input" v-model.trim="wifiPasswordAp" id="wifi_password_ap"
                 placeholder="Password"/>
          <input v-else type="password" class="input" v-model.trim="wifiPasswordAp" id="wifi_password_ap"
                 placeholder="Password"/>
          <label class="checkbox">
            <input type="checkbox" v-model="passwordClearTextAp"/> Show password
          </label>
        </p>
        <span class="help">Required.</span>
        <label class="label" for="ip_ap">IP</label>
        <p class="control">
          <input v-model.trim="ipAp" class="input" type="text" id="ip_ap" placeholder="192.168.128.1" required/>
          <span class="help">Required.</span>
        </p>
        <label class="label" for="http_port_ap">HTTP Port</label>
        <p class="control">
          <input v-model.trim="httpPortAp" class="input" type="text" id="http_port_ap" placeholder="8080" required/>
          <span class="help">Required.</span>
        </p>
        <p class="control">
        <label class="checkbox">
          <input v-model="json" :value="true" type="checkbox" /> JSON
        </label>
          <label class="checkbox">
          <input v-model="html" :value="true" type="checkbox" /> HTTP
        </label>
      </p>
        <br/>
      </div>

      <div v-if="client_mode">
        <p>
      Select the Wi-Fi network to connect to:
    </p>
        <label class="label" for="wifi_network">Network</label>
      <p class="control">
        <span class="select">
          <select v-model="ssid_select" id="wifi_network">
            <option value="select">Select...</option>
            <option v-for="network in formattedNetworks" :value="network">
              {{ network.ssid }} - {{ network.encryption }} ({{ network.signalQuality }}%)
            </option>
            <option value="other">Other/hidden network</option>
          </select>
        </span>
        <br/>
        <br/>
      </p>

      <div v-if="ssid_select === 'other'">
        <label class="label" for="wifi_ssid">Wi-Fi SSID</label>
        <p class="control">
          <input class="input" v-model.trim="ssid_input" id="wifi_ssid" type="text" placeholder="SSID" maxLength="32"
                 required/>
          <span class="help">Required.</span>
        </p>
      </div>

      <div v-if="typeof ssid_select === 'object' && ssid_select.encryption !== 'Open' || ssid_select === 'other'">
        <label class="label" for="wifi_password">Wi-Fi password</label>
        <p class="control is-grouped">
          <!-- v-model does not support dynamic :type -->
          <input v-if="passwordClearText" type="text" class="input" v-model.trim="password" id="wifi_password"
                 placeholder="Password" maxLength="64"/>
          <input v-else type="password" class="input" v-model.trim="password" id="wifi_password" placeholder="Password"
                 maxLength="64"/>
          <label class="checkbox">
            <input type="checkbox" v-model="passwordClearText"/> Show password
          </label>
        </p>
        <span class="help">Optional. Leave blank if open network.</span>
        <br/>
      </div>

      </div>

      <p class="control">
        <button type="submit" :disabled="!formIsValid" class="button is-primary">Next</button>
      </p>
    </form>
  </span>
</template>

<script>
  import axios from 'axios'
  import {API_URL} from '../../constants'

  export default {
    data() {
      return {
        loading: true,
        apiResponse: null,
        ssid_select: 'select',
        ssid_input: null,
        password: null,
        passwordClearText: false,
        client_mode: true,
        passwordClearTextAp:false,
        wifiPasswordAp:null,
        ipAp:null,
        httpPortAp:null,
        json:true,
        html:true,
        wifiSsidAp:null
      }
    },
    computed: {
      formattedNetworks: function () {
        let networks = this.apiResponse.networks.slice(0)
        networks.sort(function (networkA, networkB) {
          if (networkA.rssi > networkB.rssi) {
            return -1
          } else if (networkA.rssi < networkB.rssi) {
            return 1
          } else {
            return 0
          }
        })

        networks = networks.map(function (network) {
          if (network.rssi <= -100) {
            network.signalQuality = 0
          } else if (network.rssi >= -50) {
            network.signalQuality = 100
          } else {
            network.signalQuality = 2 * (network.rssi + 100)
          }

          switch (network.encryption) {
            case 'wep':
              network.encryption = 'WEP'
              break
            case 'wpa':
              network.encryption = 'WPA'
              break
            case 'wpa2':
              network.encryption = 'WPA2'
              break
            case 'none':
              network.encryption = 'Open'
              break
            case 'auto':
              network.encryption = 'Automatic'
              break
          }
          return network
        })

        return networks
      },
      computedSsid: function () {
        if (typeof this.ssid_select === 'object') return this.ssid_select.ssid
        else if (this.ssid_select === 'other') return this.ssid_input

        return null
      },
      formIsValid: function () {
        const needPassword = typeof this.ssid_select === 'object' && this.ssid_select.encryption !== 'Open'
        return  ((this.computedSsid && ((!needPassword || this.password) && this.client_mode)) || (!this.client_mode && this.wifiSsidAp
        &&this.wifiPasswordAp && this.ipAp &&this.httpPortAp &&(this.json||this.html )))
      }
    },
    mounted() {
      this.$emit('loading', 'Gathering available Wi-Fi networks...')
      axios.get(`${API_URL}/networks`).then((res) => {
        this.$emit('loaded')
        this.apiResponse = res.data
        this.loading = false
      }).catch(() => {
      })
    },
    methods: {
      sendDone: function () {
        if (this.client_mode) {
          this.$emit('wifiConfig', {
            mode:"client",
            ssid: this.computedSsid,
            password: typeof this.ssid_select === 'object' && this.ssid_select.encryption === 'Open' ? null : this.password
          })
          this.$emit('done')
        }
        else{
          this.$emit('wifiConfig', {
            mode:"ap",
            ssid: this.wifiSsidAp,
            password: typeof this.ssid_select === 'object' && this.ssid_select.encryption === 'Open' ? null : this.wifiPasswordAp,
            ip: this.ipAp,
            port:this.httpPortAp,
            json:this.json,
            html:this.html
          })
          this.$emit('doneGoToSendingStep')
        }


      }
    }
  }
</script>
