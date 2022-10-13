<script setup lang="ts">
import {authService} from "@/services/directus.service";
import {
  IonButton,
  IonSegment,
  IonSegmentButton,
  IonToolbar,
  IonHeader,
  IonBackButton,
  IonTitle,
  IonContent,
  IonInput,
  IonItem,
  IonLabel,
  IonList,
  IonListHeader,
  IonPage,
  IonToggle,
  toastController
} from '@ionic/vue';
import { ref } from 'vue';
import {useRouter} from "vue-router";


const router = useRouter();
/* State */

let mode = ref(false)
// The user can toggle between login and register mode in the form to show/hide additional fields
const inRegisterMode = ref(false);

// Provides two-way data binding between Vue and the input fields in the form
const userDetails = ref({
  firstName: '',
  email: '',
  password: ''
});

const login = async () => {
  try {
    await authService.login(userDetails.value.email, userDetails.value.password)
    await router.replace('/home');
  }catch (error){
    console.error(error)
  }
}


const register = async () => {
  try {
  await authService.register(userDetails.value.firstName, userDetails.value.email, userDetails.value.password);
  await login();
  /* if (wasRegistered){
     await login();
   }*/
  }catch (e) {
    await (await toastController.create({
      message: `${e}`,
      duration: 3000,
      color: "warning"
    })).present();
  }
}

const segmentChanged = (e) => {
  if (e == "login"){
    mode.value = false
  }else if (e == "register") {
    mode.value = true
  }
}
</script>

<template>
  <ion-page>
    <ion-content>
      <ion-list>
        <ion-header>
          <ion-toolbar>
            <ion-segment @ion-change="mode = !mode">
              <ion-segment-button value="login">
                <ion-label>Login</ion-label>
              </ion-segment-button>
              <ion-segment-button value="register">
                <ion-label>Registrer deg</ion-label>
              </ion-segment-button>
            </ion-segment>
          </ion-toolbar>
        </ion-header>

        <img class="hero-image" src="/assets/camera.png" />

        <ion-item lines="none">
          <ion-label class="label-mild">Ny bruker?</ion-label>
          <ion-toggle color="dark" @ion-change="inRegisterMode = !inRegisterMode"></ion-toggle>
        </ion-item>

        <hr />


        <ion-item v-if="mode">
          <ion-label class="label-mild" position="floating">Fornavn</ion-label>
          <ion-input v-model="userDetails.firstName"></ion-input>
        </ion-item>

        <ion-item>
          <ion-label class="label-mild" position="floating">Email</ion-label>
          <ion-input type="email" v-model="userDetails.email"></ion-input>
        </ion-item>

        <ion-item>
          <ion-label class="label-mild" position="floating">Passord</ion-label>
          <ion-input type="password" v-model="userDetails.password"></ion-input>
        </ion-item>

        <ion-button v-if="mode" @click="register" class="button-auth" fill="solid" color="dark" size="default">
          Registrer deg 🏕
        </ion-button>

        <ion-button v-else @click="login" class="button-auth" fill="solid" color="dark" size="default">
          Logg inn 🏕
        </ion-button>

      </ion-list>

    </ion-content>
  </ion-page>
</template>

<style scoped>
ion-content {
  --ion-background-color: #f4f4f4;
  display: flex;
}

ion-list {
  display: flex;
  flex-direction: column;
}

.label-mild {
  --color: #8a8a8a !important;
}

.hero-image {
  width: 80vw;
  align-self: center;
}

.button-auth {
  margin-top: 50px;
  margin-left: 10px;
  margin-right: 10px;
}
</style>