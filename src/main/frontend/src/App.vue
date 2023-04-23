<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

<div v-else>
    <button @click="registering = false"
            :class="registering ? 'button-outline' : ''">Logowanie</button>
    <button @click="registering = true"
            :class="!registering ? 'button-outline' : ''">Rejestracja</button>
  <div v-if ="message" :class="message.includes('Nie udało się') ? 'alert alert2' : 'alert'">{{  message }}</div>
    <LoginForm v-if="!registering" @login="(user) => logMeIn(user)"></LoginForm>
    <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>
    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";


export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      registering: false,
      authenticatedUsername: '',
      message: '',
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            const token = response.data.token;
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
            this.authenticatedUsername = user.login;
            axios.get('/api/meetings').then(response => console.log(response.data));
          })
          .catch(response => {
            this.message = 'Nie udało się zalogować'
          })
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },
    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            this.message = 'Twoje konto zostało założone'
          })
          .catch(response => {
            this.message = 'Nie udało się założyć konta. Spróbuj ponownie'
          })

    }
  }

}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}
.alert{
  border: 2px green solid;
  background: lightgreen;
}
.alert2{
  border: 2px red solid;
  background: lightcoral;
}
</style>
