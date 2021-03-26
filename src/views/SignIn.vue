<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <amplify-authenticator>
          <div v-if="authState === 'signedin' && user">
            <div>Hello, {{ user.username }}</div>
          </div>
          <amplify-sign-up
            slot="sign-up"
            :form-fields.prop="signUpFormFields"
          ></amplify-sign-up>
          <amplify-sign-out></amplify-sign-out>
        </amplify-authenticator>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { onAuthUIStateChange } from '@aws-amplify/ui-components'

export default {
  name: 'AuthStateApp',
  created() {
    this.unsubscribeAuth = onAuthUIStateChange((authState, authData) => {
      this.authState = authState
      this.user = authData
    })
  },
  data() {
    return {
      user: undefined,
      authState: undefined,
      unsubscribeAuth: undefined,
      signUpFormFields: [
        {
          type: 'username',
          required: true,
        },
        {
          type: 'email',
          required: true,
        },
        {
          type: 'password',
          required: true,
        },
      ],
    }
  },
  beforeDestroy() {
    this.unsubscribeAuth()
  },
}
</script>

<style></style>
