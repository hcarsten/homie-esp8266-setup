<template>
  <span>

     <p class="control">
        <label class="checkbox">
          <input v-model="cumuloscityEnabled" type="radio" :value="false"/> Standard MQTT Mode
        </label>
        <br/>
        <label class="checkbox">
          <input v-model="cumuloscityEnabled" type="radio" :value="true"/> Cumuloscity MQTT Mode
        </label>
      </p>

    <p>
      Enter the MQTT credentials.
    </p>

    <form @submit.prevent="sendDone">
      <div v-if="!cumuloscityEnabled">
      <label class="label" for="mqtt_broker_address">MQTT broker address</label>
      <p class="control">
        <input v-model.trim="host" class="input" type="text" id="mqtt_broker_address" placeholder="IP or hostname"
               pattern="^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$|^(([a-zA-Z0-9]|[a-zA-Z0-9][a-zA-Z0-9\-]*[a-zA-Z0-9])\.)*([A-Za-z0-9]|[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9])$"
               required/>
        <span class="help">Required.</span>
      </p>

      <label class="label" for="mqtt_broker_port">MQTT broker port</label>
      <p class="control">
        <input v-model.number="port" class="input" type="number" step="1" min="1" max="65535" id="mqtt_broker_port"
               placeholder="Port number" required/>
        <span class="help">Required.</span>
      </p>

      <label class="label" for="homie_base_topic">Homie base topic</label>
      <p class="control">
        <input v-model.trim="baseTopic" class="input" type="text" id="homie_base_topic" placeholder="Base topic"/>
        <span class="help">Optional. The default value is <span class="tag">homie/</span>.</span>
      </p>

      <p class="control">
        <label class="checkbox">
          <input v-model="auth" type="checkbox"/> Use MQTT authentication
        </label>
      </p>

      <div v-if="auth">
        <label class="label" for="mqtt_username">MQTT username</label>
        <p class="control">
          <input v-model.trim="username" class="input" type="text" id="mqtt_username" placeholder="Username" required/>
          <span class="help">Required.</span>
        </p>

        <label class="label" for="mqtt_password">MQTT password</label>
        <p class="control is-grouped">
          <!-- v-model does not support dynamic :type -->
          <input v-if="passwordClearText" type="text" class="input" v-model.trim="password" id="mqtt_password"
                 placeholder="Password"/>
          <input v-else type="password" class="input" v-model.trim="password" id="mqtt_password"
                 placeholder="Password"/>
          <label class="checkbox">
            <input type="checkbox" v-model="passwordClearText"/> Show password
          </label>
        </p>
        <span class="help">Required.</span>

        <br/>
      </div>
        </div>
      <div v-if="cumuloscityEnabled">
        <label class="label" for="cumulocity_tenant">Tenant</label>
      <p class="control">
        <input v-model="cumuloscity.tenant" class="input"  id="cumulocity_tenant"
               placeholder="Enter Tenant" required/>
        <span class="help">Required.</span>
      </p>
        <label class="label" for="cumulocity_clientid">ClientID</label>
        <p class="control">
        <input v-model="cumuloscity.clientid" class="input"  id="cumulocity_clientid"
               placeholder="Enter ClientId" required/>
        <span class="help">Required.</span>
      </p>

        <label class="label" for="cumulocity_username">Username</label>
        <p class="control">
        <input v-model="cumuloscity.username" class="input"  id="cumulocity_username"
               placeholder="Enter Username" required/>
        <span class="help">Required.</span>
      </p>
        <label class="label" for="cumulocity_password">Password</label>
        <p class="control is-grouped">
          <!-- v-model does not support dynamic :type -->
          <input v-if="passwordClearText" type="text" class="input" v-model.trim="cumuloscity.password" id="cumulocity_password"
                 placeholder="Password"/>
          <input v-else type="password" class="input" v-model.trim="cumuloscity.password" id="cumulocity_password"
                 placeholder="Password"/>

          <label class="checkbox">
            <input type="checkbox" v-model="passwordClearText"/> Show password
          </label>
        </p>
        <span class="help">Required.</span>
      </div>
      <p class="control">
        <button type="submit" :disabled="!formIsValid" class="button is-primary">Next</button>
      </p>
    </form>
  </span>
</template>

<script>
  export default {
    data() {
      return {
        cumuloscityEnabled: false,
        passwordClearText: false,
        host: null,
        port: 1883,
        baseTopic: null,
        auth: false,
        username: null,
        password: null,
        cumuloscity:{
          username:null,
          password:null,
          tenant:null,
          clientid:null

        }
      }
    },
    computed: {
      formIsValid: function () {
        if (!this.cumuloscityEnabled) {
          if (!this.host) return false
          if (!this.port) return false

          if (this.auth) {
            if (!this.username) return false
            if (!this.password) return false
          }
        }
        if (this.cumuloscityEnabled){
          if (!this.cumuloscity.tenant) return false;
          if (!this.cumuloscity.clientid) return false;
          if (!this.cumuloscity.password) return false;
          if (!this.cumuloscity.username) return false;
        }
        return true
      }
    },
    methods: {
      sendDone: function () {
        const mqtt = {}
        if (!this.cumuloscityEnabled){
          mqtt.host = this.host
          mqtt.port = this.port
          mqtt.cumuloscity=false
          if (this.baseTopic) mqtt['base_topic'] = this.baseTopic

          mqtt.auth = false
          if (this.auth) {
            mqtt.auth = true
            mqtt.username = this.username
            mqtt.password = this.password
          }
        }
        if(this.cumuloscityEnabled){
          mqtt.host="mqtt.cumulocity.com"
          mqtt.password= this.cumuloscity.password
          mqtt.username=this.cumuloscity.username
          mqtt.tenant= this.cumuloscity.tenant
          mqtt.clientid=this.cumuloscity.clientid
          mqtt.cumuloscity=true
        }

        this.$emit('mqttConfig', mqtt)
        this.$emit('done')
      }
    }
  }
</script>
