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
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Photo Gallery</ion-title>
        </ion-toolbar>
      </ion-header>
    </ion-content>

    <ion-fab vertical="bottom" horizontal="center" slot="fixed">
      <ion-fab-button>
        <ion-icon :icon="camera"></ion-icon>
      </ion-fab-button>
    </ion-fab>
  </ion-page>
</template>


<script setup lang="ts">
import { camera } from 'ionicons/icons';
import { IonPage, IonHeader, IonFab, IonFabButton, IonIcon, IonToolbar, IonTitle, IonContent } from '@ionic/vue';

// CHANGE: Add `usePhotoGallery` import
import { usePhotoGallery } from '@/composables/usePhotoGallery';

// CHANGE: Destructure `addNewToGallery` from `usePhotoGallery()
const { addNewToGallery } = usePhotoGallery();
</script>
