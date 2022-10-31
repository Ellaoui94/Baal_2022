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

const campingSpot = ref({})
const comments = ref([])

interface ICampingSpot {
  camping_spots_by_id: {
    title: string,
    description: string,
    hashtags: string[],
    image: {
      id: number
    },
    comment_fk: {
      id: number,
      first_name: string,
      comment: string,
      date_created: string,
      user_created: {
        first_name: string,
        avatar: {
          id : number
        }
      },
    }
  }
}

const loadSpot = async () => {
  const response = await directus.graphql.items<ICampingSpot>(`
query MyQuery {
  camping_spots_by_id(id: ${id}) {
    title
    description
    hashtags
    image {
      id
    }
    comment_fk {
      id
      first_name
      comment
      date_created
      user_created {
        first_name
        avatar {
          id
        }
      }
    }
  }
}
`)
  const responseData = response.data as ICampingSpot

  if (response.status === 200 && responseData) {
    campingSpot.value = responseData.camping_spots_by_id;
    comments.value = responseData.camping_spots_by_id.comment_fk
  }
}

onIonViewDidEnter(async () => {
  await loadSpot()
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

const addNewComment = async () => {
  if (newCommentText.value) {
    try {
      await directus.items('campSpot_comments').createOne({
        comment: newCommentText.value,
        campSpot_fk: id,
      })
    } catch (e) {
      console.error(e)
    }
  }

  newCommentText.value = "";
  isModalOpen.value = false;
  await loadSpot();
}
</script>

<template>
  <ion-page>
    <ion-header v-if="!campingSpot.image" :translucent="true">
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
        <ion-title>{{ campingSpot.title }} ({{ id }})</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="isModalOpen = true">
            <ion-icon :icon="chatboxOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content v-if="campingSpot.image" :fullscreen="true">
      <camping-spot-image :image-id="campingSpot.image.id"/>
      <ion-chip v-for="hashtag in campingSpot.hashtags" :key="hashtag" color="tertiary">#{{ hashtag }}</ion-chip>

      <ion-card>
        <ion-card-title>{{ campingSpot.title }}</ion-card-title>
        <ion-card-content>
          {{ campingSpot.description }}
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
          <ion-item v-for="comment in comments" :key="comment.id" lines="none">
            <ion-avatar slot="start">
              <camping-spot-image v-if="!comment.user_created.avatar" :image-id="campingSpot.image.id"/>
              <camping-spot-image v-else :image-id="comment.user_created.avatar.id"/>
            </ion-avatar>
            <ion-label class="ion-text-wrap">
              <ion-header>
                <b>{{ comment.user_created.first_name }}</b>
              </ion-header>
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