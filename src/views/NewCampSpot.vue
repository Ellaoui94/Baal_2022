<script setup lang="ts">
import {
  IonBackButton,
  IonButton,
  IonButtons,
  IonChip,
  IonContent,
  IonHeader,
  IonInput,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTextarea,
  IonTitle,
  IonToolbar
} from "@ionic/vue";
import {ref} from "vue";
import {Camera, CameraResultType} from "@capacitor/camera";
import {directus} from "@/services/directus.service";

const newHashTag = ref("")
const newCampSpot = ref({
  title: "",
  description: "",
  hashtags: [],
  image: "",
});

const addNewHashtag = () => {
  if (newHashTag.value){
    newCampSpot.value.hashtags.push(newHashTag.value)
    newHashTag.value = "";
  }
}


const postNewCampSpot = async () => {
  if (!newCampSpot.value.image) {
    alert("BILDE REQUIRED")
    return;
  }

  try {
    const res = await fetch(newCampSpot.value.image)
    const imgBlob = await res.blob();

    const formData = new FormData();
    formData.append('file', imgBlob);
    const fileUpload = await directus.files.createOne(formData)

    if (fileUpload){
      await directus.items('camping_spots').createOne({
        title: newCampSpot.value.title,
        description: newCampSpot.value.description,
        hashtags: newCampSpot.value.hashtags,
        image: fileUpload.id
      })
    }
  }catch (e) {
    console.error(e)
  }

}

const openCamera = async () => {
  const pic = await Camera.getPhoto({
    quality: 100,
    allowEditing: false,
    resultType: CameraResultType.Uri
  })

  if (pic.webPath) {
    newCampSpot.value.image = pic.webPath
  }
}
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button deafult-href="/"></ion-back-button>
        </ion-buttons>
        <ion-title>Baal ⛺</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-list>
        <ion-button v-if="!newCampSpot.image" @click="openCamera" color="light" class="imageBtn">Velg fil 🖼 eller ta et bilde 📸</ion-button>

        <img v-if="newCampSpot.image" @click="openCamera" :src="newCampSpot.image"/>

        <ion-item>
          <ion-label position="floating">Tittel</ion-label>
          <ion-input type="text" v-model="newCampSpot.title"></ion-input>
        </ion-item>

        <ion-item>
          <ion-label position="floating">Description</ion-label>
          <ion-textarea v-model="newCampSpot.description"></ion-textarea>
        </ion-item>

        <ion-item>
          <ion-label position="floating">Ny tag</ion-label>
          <ion-input v-model="newHashTag"></ion-input>

          <ion-button slot="end" size="default" @click="addNewHashtag">Legg til tag</ion-button>
        </ion-item>

        <ion-item lines="none">
          <ion-chip color="primary" v-for="tag in newCampSpot.hashtags" :key="tag">{{tag}}</ion-chip>
        </ion-item>

        <ion-button @click="postNewCampSpot">Legg til spot 🏕</ion-button>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<style>
.imageBtn {
  margin: 10px;
  height: 20vh;
  border: 2px #8a8a8a dashed;
  border-radius: 8px;
  font-size: medium;
}

ion-content {
  display: flex;
}

ion-list {
  display: flex;
  flex-direction: column;
}

.postBtn {

}
</style>