<script setup lang="ts">
import {
  IonButton,
  IonButtons,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardSubtitle,
  IonCardTitle,
  IonContent,
  IonHeader,
  IonIcon,
  IonPage,
  IonTitle,
  IonToolbar, onIonViewDidEnter, IonRefresher, IonRefresherContent
} from '@ionic/vue';
import {addCircleOutline, chevronDownCircleOutline  } from "ionicons/icons";
import {ref} from "vue";
import {directus} from "@/services/directus.service";
import CampingSpotCard from "@/components/CampingSpotCard.vue";

const campingSpots = ref([])

//Siden responesn er flere enn en, så ha med array[]
interface ICampingSpotS{
  camping_spots:[ {
    id: number,
    title: string,
    description: string,
    hashtags: string[],
    image: {
      id: number
    },
    user_created: {
      first_name: string
    },
  }]
}

onIonViewDidEnter(async () => {
  await testFun()
})

const testFun = async () => {
  const response = await directus.graphql.items<ICampingSpotS>(`
query MyQuery {
  camping_spots {
    id
    title
    description
    hashtags
    image {
      id
    }
    user_created {
      first_name
    }
  }
}
`)

  const responseData = response.data as ICampingSpotS

  if (response.status === 200 && responseData) {
    campingSpots.value = [...responseData.camping_spots];
  }

}
/*let id = 1;

const campingSpots = ref([
  {
    id: id++,
    title: "Fin plass ved Ulsrudvann",
    description: "Fant den en vakker dag",
    hashtags: "#camping #oslo",
    imageURL: "https://images.squarespace-cdn.com/content/v1/587642149f7456eeebf28e13/1560419927810-JZX1PLQ34SLZ95GY3TF3/Telttur-ved-Oslofjorden-8.jpg?format=1500w"
  },
]);

const addCampSpot = () => {
  const newSpot = {
    id: id++,
    title: "Fin plass ved Holmlia",
    description: "Fant den en normal dag",
    hashtags: "#camping #oslo #holmlia",
    imageURL: "https://i0.wp.com/tikicamp.com/wp-content/uploads/2022/06/DSC_6053.jpg?fit=2048%2C1365&ssl=1"

  }
  campingSpots.value.unshift(newSpot);

  //If I have the whole object inside unshift and spread operator, I will get different value for each of them I.E: 1,2,3. And not 1,2,2,3,3

  campingSpots.value = [newSpot, ...campingSpots.value];

  console.log(campingSpots.value)
}
  console.log(id)*/

const doRefresh = (event: CustomEvent) => {
  testFun()
  event.target.complete();
}

</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Baal ⛺</ion-title>
        <ion-buttons slot="primary">
          <ion-button router-link="/newSpot">
            <ion-icon slot="icon-only" :icon="addCircleOutline"></ion-icon>
          </ion-button>
          <ion-button href="/auth">
            <ion-icon slot="icon-only" :icon="addCircleOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-refresher slot="fixed"  pull-factor="0.5" pull-min="100" pull-max="200" @ionRefresh="doRefresh($event)">
        <ion-refresher-content :pulling-icon="chevronDownCircleOutline"
                               pulling-text="Pull to refresh"
                               refreshing-spinner="circles"
                               refreshing-text="Refreshing..."></ion-refresher-content>
      </ion-refresher>
      <camping-spot-card v-for="spot in campingSpots" :key="spot.id" :spot="spot" />
    </ion-content>
  </ion-page>
</template>

