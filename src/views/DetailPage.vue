<script setup lang="ts">
/* eslint-disable no-undef */
import {useRoute} from "vue-router";
import {
  IonBackButton,
  IonButtons,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardSubtitle,
  IonCardTitle,
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButton,
  IonIcon,
  IonChip,
  IonAvatar, IonText, IonItem, IonListHeader, IonLabel, IonList, IonSpinner, IonTextarea,
  IonModal, onIonViewDidEnter
} from "@ionic/vue";

import {chatboxOutline} from 'ionicons/icons';
import {ref} from "vue";
import {Geolocation} from '@capacitor/geolocation';
import {Loader} from "@googlemaps/js-api-loader";
import {directus} from "@/services/directus.service";
import CampingSpotImage from "@/components/CampingSpotImage.vue"

const GOOGLE_MAPS_API_KEY = 'AIzaSyDTjuG0cFQlQWCBaNmlWfxYGJ_wCg_EXKk'

const route = useRoute();
const {id} = route.params;

const isModalOpen = ref(false);
const newCommentText = ref('')

/*
const userAccessToken = localStorage.getItem('auth_token')
*/


/*const campingSpots = ref({
  id: id,
  title: "Fin plass ved Ulsrudvann",
  description: "Fant denne da jeg gikk kveldstur rundt vannet",
  imageURL: "https://www.skiforeningen.no/globalassets/bilder/hege-w/ukens-tur/telttur/telt-nedreblanksjo.jpg?w=1200",
  hashtags: ["østmarka", "oslo", "nærtur"],
  location: {
    latitude: 59.888033,
    longitude: 10.862266
  },
  comments: [
    {
      id: 1,
      username: "Andreas",
      comment: "Flott ved Ulsrud, men ta med drikkevann!"
    }
  ]
})*/

const campingSpots = ref({})

onIonViewDidEnter(async () => {
  const response = await directus.graphql.items(`
query MyQuery {
  camping_spots_by_id(id: ${id}) {
    title
    description
    image {
      id
    }
  }
}
`)

  if (response.status === 200 && response.data) {
    campingSpots.value = response.data.camping_spots_by_id;
    console.log(campingSpots.value)
  }

})


//------------------------------------------------------------------------
const printCurrentPosition = async () => {
  const coordinates = await Geolocation.getCurrentPosition();

  return coordinates.coords
};

let latitude = null;
let longitude;
printCurrentPosition().then((s) => {
  latitude = s.latitude
  longitude = s.longitude
});

const loader = new Loader({apiKey: GOOGLE_MAPS_API_KEY})
const mapDiv = ref(null)
console.log(latitude)
//------------------------------------------------------------------------


isModalOpen.value = false;
newCommentText.value = '';

const addNewComment = () => {
  campingSpots.value.comments.unshift({
    id: 2,
    username: "N/A",
    comment: newCommentText.value
  })
}

console.log(campingSpots)

</script>

<template>
  <ion-page>
    <ion-header v-if="!campingSpots.image" :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button deafult-href="/"></ion-back-button>
        </ion-buttons>
        <ion-spinner style="margin-left: 30px" name="circular"></ion-spinner>
      </ion-toolbar>
    </ion-header>

    <ion-header v-else :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button deafult-href="/"></ion-back-button>
        </ion-buttons>
        <ion-title>{{ campingSpots.title }} ({{ id }})</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="isModalOpen = true">
            <ion-icon :icon="chatboxOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content v-if="campingSpots.image" :fullscreen="true">
      <camping-spot-image :image-id="campingSpots.image.id"/>
        <ion-chip v-for="hashtag in campingSpots.hashtags" :key="hashtag" color="tertiary">#{{ hashtag }}</ion-chip>

        <ion-card>
          <ion-card-header>
            <ion-card-subtitle v-for="tag in campingSpots.hashtags" :key="tag" style="display: inline-block">
              #{{ tag }}
            </ion-card-subtitle>
            <ion-card-subtitle>Turbeskrivelse</ion-card-subtitle>
          </ion-card-header>

          <ion-card-title>{{ campingSpots.title }}</ion-card-title>
          <ion-card-content>
            {{ campingSpots.description }}
          </ion-card-content>
        </ion-card>

        <ion-card>
          <ion-list>
            <ion-list-header>
              <ion-label>
                Er du en hater eller en lover? Vis deg frem 😈
                <ion-icon :icon="chatboxOutline"></ion-icon>
              </ion-label>
            </ion-list-header>
            <ion-item v-for="comment in campingSpots.comments" :key="comment.id" lines="none">
              <ion-avatar slot="start">
                <camping-spot-image :image-id="campingSpots.image.id"/>
              </ion-avatar>
              <ion-label class="ion-text-wrap">
                <ion-text>
                  <b>{{ comment.username }}</b>
                </ion-text>
                <ion-text>
                  <b>{{ comment.comment }}</b>
                </ion-text>
              </ion-label>
            </ion-item>
          </ion-list>
        </ion-card>

        <ion-modal
            :is-open="isModalOpen"
            :initial-breakpoint="0.25"
            :breakpoints="[0, 0.25, 0.5, 0.75]"
            @did-dismiss="isModalOpen = false">
          <ion-content>
            <ion-item lines="none">
              <ion-label position="floating">Ny hate eller love?</ion-label>
              <ion-textarea v-model="newCommentText"></ion-textarea>
              <ion-button @click="addNewComment"></ion-button>
            </ion-item>
          </ion-content>
        </ion-modal>
    </ion-content>
  </ion-page>
</template>