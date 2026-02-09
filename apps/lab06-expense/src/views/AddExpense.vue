<template>
  <ion-page>
    <ion-header class="custom-header">
      <ion-toolbar color="primary">
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/tab1" text=""></ion-back-button>
        </ion-buttons>
        <ion-title class="page-title">เพิ่มรายการ</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding custom-content">
      <div class="form-container">
        <ion-item class="custom-item" lines="none">
          <ion-input 
            label="ชื่อรายการ" 
            label-placement="floating"
            v-model="title"
            placeholder="เช่น ค่าอาหาร, เงินเดือน"
            class="custom-input">
          </ion-input>
        </ion-item>

        <ion-item class="custom-item" lines="none">
          <ion-input
            label="จำนวนเงิน"
            label-placement="floating"
            type="number"
            v-model="amount"
            placeholder="0.00"
            class="custom-input">
          </ion-input>
        </ion-item>

        <ion-item class="custom-item" lines="none">
          <ion-select 
            label="ประเภท" 
            label-placement="floating"
            v-model="type"
            class="custom-select"
            interface="action-sheet">
            <ion-select-option value="income">💰 รายรับ</ion-select-option>
            <ion-select-option value="expense">💸 รายจ่าย</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item class="custom-item" lines="none">
          <ion-select 
            label="หมวดหมู่" 
            label-placement="floating"
            v-model="category"
            class="custom-select"
            interface="action-sheet">
            <ion-select-option value="food">🍔 อาหาร</ion-select-option>
            <ion-select-option value="transport">🚗 ค่าเดินทาง</ion-select-option>
            <ion-select-option value="shopping">🛍️ ช้อปปิ้ง</ion-select-option>
            <ion-select-option value="entertainment">🎬 ความบันเทิง</ion-select-option>
            <ion-select-option value="bills">💡 ค่าน้ำค่าไฟ</ion-select-option>
            <ion-select-option value="salary">💼 เงินเดือน</ion-select-option>
            <ion-select-option value="other">📌 อื่นๆ</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item class="custom-item" lines="none">
          <ion-textarea 
            label="หมายเหตุ" 
            label-placement="floating"
            v-model="note"
            :rows="3"
            placeholder="บันทึกเพิ่มเติม (ถ้ามี)"
            class="custom-textarea">
          </ion-textarea>
        </ion-item>

        <ion-button 
          expand="block" 
          @click="saveExpense"
          :disabled="!title || !amount"
          class="save-button">
          <ion-icon slot="start" :icon="saveOutline"></ion-icon>
          บันทึกข้อมูล
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { 
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonItem, IonInput, IonSelect, IonSelectOption, 
  IonTextarea, IonButton, IonButtons, IonBackButton,
  IonIcon, useIonRouter, toastController
} from "@ionic/vue";
import { collection, addDoc } from "firebase/firestore";
import { db } from "@/firebase";
import { saveOutline } from "ionicons/icons";

const router = useIonRouter();

const title = ref("");
const amount = ref<number>(0);
const type = ref("expense");
const category = ref("food");
const note = ref("");

const saveExpense = async () => {
  try {
    await addDoc(collection(db, "expenses"), {
      title: title.value,
      amount: Number(amount.value),
      type: type.value,
      category: category.value,
      note: note.value,
      createdAt: new Date()
    });

    const toast = await toastController.create({
      message: 'บันทึกข้อมูลเรียบร้อย',
      duration: 2000,
      color: 'success',
      position: 'top'
    });
    await toast.present();

    router.push("/tabs/tab1");
  } catch (error) {
    console.error("Error adding document: ", error);
    
    const toast = await toastController.create({
      message: 'เกิดข้อผิดพลาดในการบันทึก',
      duration: 2000,
      color: 'danger',
      position: 'top'
    });
    await toast.present();
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&display=swap');

* {
  font-family: 'Kanit', sans-serif;
}

.custom-header ion-toolbar {
  --background: linear-gradient(135deg, #e46c32 0%, #5eb298 100%);
  --color: white;
}

.page-title {
  font-weight: 600;
  font-size: 1.2rem;
  letter-spacing: 0.5px;
}

.custom-content {
  --background: linear-gradient(180deg, #f8f9fa 0%, #ffffff 100%);
}

.form-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px 0;
}

.custom-item {
  --background: white;
  --border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(228, 108, 50, 0.1);
  border-left: 4px solid #e46c32;
  transition: all 0.3s ease;
}

.custom-item:hover {
  box-shadow: 0 4px 12px rgba(228, 108, 50, 0.2);
  transform: translateY(-2px);
}

.custom-item::part(native) {
  padding: 16px;
}

.custom-input,
.custom-select,
.custom-textarea {
  font-family: 'Kanit', sans-serif;
  --color: #5f4256;
  --placeholder-color: #b0b0b0;
  --placeholder-opacity: 0.7;
}

ion-label {
  font-family: 'Kanit', sans-serif !important;
  font-weight: 500 !important;
  color: #6e7aaa !important;
}

.save-button {
  --background: linear-gradient(135deg, #e46c32 0%, #5eb298 100%);
  --background-hover: linear-gradient(135deg, #d45a28 0%, #4da085 100%);
  --background-activated: linear-gradient(135deg, #c24e1e 0%, #3d8f72 100%);
  --border-radius: 12px;
  --box-shadow: 0 4px 12px rgba(228, 108, 50, 0.3);
  margin-top: 24px;
  height: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-transform: none;
  transition: all 0.3s ease;
}

.save-button:hover {
  --box-shadow: 0 6px 16px rgba(228, 108, 50, 0.4);
  transform: translateY(-2px);
}

.save-button::part(native) {
  font-family: 'Kanit', sans-serif;
}

.save-button[disabled] {
  --background: #d0d0d0;
  --box-shadow: none;
  opacity: 0.6;
}

ion-select-option {
  font-family: 'Kanit', sans-serif;
}

/* Animation for items */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.custom-item {
  animation: fadeInUp 0.4s ease-out;
}

.custom-item:nth-child(1) { animation-delay: 0.1s; }
.custom-item:nth-child(2) { animation-delay: 0.2s; }
.custom-item:nth-child(3) { animation-delay: 0.3s; }
.custom-item:nth-child(4) { animation-delay: 0.4s; }
.custom-item:nth-child(5) { animation-delay: 0.5s; }
</style>