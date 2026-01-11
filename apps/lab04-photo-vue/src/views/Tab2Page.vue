<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          Photo Gallery <br/>
          โดย วิสิษฎ์ ศรีอดิศักดิ์ รหัส 663380234-8
        </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <!-- Photo Grid -->
      <ion-grid>
        <ion-row>
          <ion-col size="6" :key="photo.filepath" v-for="photo in photos">
            <ion-img :src="photo.webviewPath"></ion-img>
          </ion-col>
        </ion-row>
      </ion-grid>

      <!-- Floating Action Button -->
      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button @click="addNewToGallery()">
          <ion-icon :icon="camera"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { camera, trash, close } from 'ionicons/icons';
import {
  IonPage,
  IonHeader,
  IonFab,
  IonFabButton,
  IonIcon,
  IonToolbar,
  IonTitle,
  IonContent,
  IonGrid,
  IonRow,
  IonCol,
  IonImg,
} from '@ionic/vue';


// CHANGE: Add `usePhotoGallery` import
import { usePhotoGallery } from '@/composables/usePhotoGallery';
// CHANGE: Add `photos` array to destructure from `usePhotoGallery()`
const { photos, addNewToGallery } = usePhotoGallery();
</script>
