<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/tab1" text=""></ion-back-button>
        </ion-buttons>
        <ion-title>แก้ไขรายการ</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="loading" class="loading-state">
        <ion-spinner></ion-spinner>
        <p>กำลังโหลดข้อมูล...</p>
      </div>

      <div v-else class="form-container">
        <ion-item class="custom-item" lines="none">
          <ion-input 
            label="ชื่อรายการ" 
            label-placement="floating"
            v-model="title"
            class="custom-input">
          </ion-input>
        </ion-item>

        <ion-item class="custom-item" lines="none">
          <ion-input
            label="จำนวนเงิน"
            label-placement="floating"
            type="number"
            v-model="amount"
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
            class="custom-textarea">
          </ion-textarea>
        </ion-item>

        <ion-button 
          expand="block" 
          @click="updateExpense"
          :disabled="!title || !amount"
          class="update-button">
          <ion-icon slot="start" :icon="saveOutline"></ion-icon>
          อัปเดตข้อมูล
        </ion-button>

        <ion-button 
          expand="block" 
          @click="confirmDelete"
          class="delete-button">
          <ion-icon slot="start" :icon="trashOutline"></ion-icon>
          ลบรายการ
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonItem, IonInput, IonSelect, IonSelectOption,
  IonTextarea, IonButton, IonButtons, IonBackButton,
  IonIcon, IonSpinner, useIonRouter,
  alertController, toastController
} from "@ionic/vue";

import { database } from "@/firebase";
import { ref as dbRef, get, update, remove } from "firebase/database";

import { saveOutline, trashOutline } from "ionicons/icons";


const route = useRoute();
const router = useIonRouter();

const loading = ref(true);
const title = ref("");
const amount = ref<number>(0);
const type = ref("expense");
const category = ref("food");
const note = ref("");

const expenseId = route.params.id as string;

// โหลดข้อมูล
onMounted(async () => {
  try {
    const snapshot = await get(
      dbRef(database, `expenses/${expenseId}`)
    );

    if (snapshot.exists()) {
      const data = snapshot.val();
      title.value = data.title;
      amount.value = data.amount;
      type.value = data.type;
      category.value = data.category;
      note.value = data.note || "";
    } else {
      const toast = await toastController.create({
        message: "ไม่พบข้อมูล",
        duration: 2000,
        color: "danger",
        position: "top"
      });
      await toast.present();
      router.push("/tabs/tab1");
    }
  } catch (error) {
    console.error("Error loading data:", error);
  } finally {
    loading.value = false;
  }
});


// อัปเดตข้อมูล
const updateExpense = async () => {
  try {
    await update(
      dbRef(database, `expenses/${expenseId}`),
      {
        title: title.value,
        amount: Number(amount.value),
        type: type.value,
        category: category.value,
        note: note.value,
        updatedAt: Date.now()
      }
    );

    const toast = await toastController.create({
      message: "อัปเดตข้อมูลเรียบร้อย",
      duration: 2000,
      color: "success",
      position: "top"
    });
    await toast.present();

    router.push("/tabs/tab1");
  } catch (error) {
    console.error("Error updating expense:", error);

    const toast = await toastController.create({
      message: "เกิดข้อผิดพลาดในการอัปเดต",
      duration: 2000,
      color: "danger",
      position: "top"
    });
    await toast.present();
  }
};


const confirmDelete = () => {
  if (confirm("ต้องการลบรายการนี้ใช่หรือไม่?")) {
    deleteExpense();
  }
};



// ลบข้อมูล
const deleteExpense = async () => {
  try {
    await remove(
      dbRef(database, `expenses/${expenseId}`)
    );

    const toast = await toastController.create({
      message: 'ลบข้อมูลเรียบร้อย',
      duration: 2000,
      color: 'success',
      position: 'top'
    });
    await toast.present();

    router.push("/tabs/tab1");
  } catch (error) {
    console.error("Error deleting expense:", error);

    const toast = await toastController.create({
      message: 'เกิดข้อผิดพลาดในการลบ',
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

ion-header ion-toolbar {
  --background: linear-gradient(135deg, #6e7aaa 0%, #5eb298 100%);
  --color: white;
}

ion-title {
  font-weight: 600;
  font-size: 1.2rem;
  letter-spacing: 0.5px;
}

ion-content {
  --background: linear-gradient(180deg, #f8f9fa 0%, #ffffff 100%);
}

/* Form Container */
.form-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px 0;
}

/* Custom Items */
.custom-item {
  --background: white;
  --border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(110, 122, 170, 0.1);
  border-left: 4px solid #6e7aaa;
  transition: all 0.3s ease;
}

.custom-item:hover {
  box-shadow: 0 4px 12px rgba(110, 122, 170, 0.2);
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

/* Buttons */
.update-button {
  --background: linear-gradient(135deg, #5eb298 0%, #4da085 100%);
  --background-hover: linear-gradient(135deg, #4da085 0%, #3d8f72 100%);
  --background-activated: linear-gradient(135deg, #3d8f72 0%, #2d7d5f 100%);
  --border-radius: 12px;
  --box-shadow: 0 4px 12px rgba(94, 178, 152, 0.3);
  margin-top: 24px;
  height: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-transform: none;
  transition: all 0.3s ease;
}

.update-button:hover {
  --box-shadow: 0 6px 16px rgba(94, 178, 152, 0.4);
  transform: translateY(-2px);
}

.update-button::part(native) {
  font-family: 'Kanit', sans-serif;
}

.update-button[disabled] {
  --background: #d0d0d0;
  --box-shadow: none;
  opacity: 0.6;
}

.delete-button {
  --background: transparent;
  --background-hover: rgba(178, 67, 66, 0.1);
  --background-activated: rgba(178, 67, 66, 0.2);
  --border-color: #b24342;
  --border-width: 2px;
  --border-style: solid;
  --border-radius: 12px;
  --color: #b24342;
  --box-shadow: 0 2px 8px rgba(178, 67, 66, 0.2);
  margin-top: 16px;
  height: 50px;
  font-size: 1.05rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-transform: none;
  transition: all 0.3s ease;
}

.delete-button:hover {
  --box-shadow: 0 4px 12px rgba(178, 67, 66, 0.3);
  --background: rgba(178, 67, 66, 0.05);
  transform: translateY(-2px);
}

.delete-button::part(native) {
  font-family: 'Kanit', sans-serif;
}

/* Loading State */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60vh;
  color: #6e7aaa;
}

.loading-state ion-spinner {
  --color: #e46c32;
  width: 50px;
  height: 50px;
  margin-bottom: 20px;
}

.loading-state p {
  margin-top: 15px;
  font-size: 1.1rem;
  font-weight: 500;
  color: #5f4256;
}

/* Select Options */
ion-select-option {
  font-family: 'Kanit', sans-serif;
}

/* Animation */
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

.update-button {
  animation: fadeInUp 0.5s ease-out;
  animation-delay: 0.6s;
}

.delete-button {
  animation: fadeInUp 0.5s ease-out;
  animation-delay: 0.7s;
}

/* Responsive */
@media (min-width: 768px) {
  .form-container {
    padding: 30px 20px;
  }
}
</style>