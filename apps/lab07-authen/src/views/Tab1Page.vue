<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>โปรไฟล์</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="handleLogout">
            <ion-icon :icon="logOutOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="user-content">
      <div class="user-container" v-if="user">
        <!-- Profile Header -->
        <div class="profile-header">
          <div class="profile-avatar">
            <img v-if="user.photoUrl" :src="user.photoUrl" alt="Profile" />
            <ion-icon v-else :icon="personCircle" class="default-avatar"></ion-icon>
          </div>
          <h2 class="profile-name">{{ user.displayName || 'ผู้ใช้งาน' }}</h2>
          <p class="profile-subtitle">ยินดีต้อนรับกลับมา</p>
        </div>

        <!-- User Info Cards -->
        <div class="info-cards">
          <div class="info-card" v-if="user.email">
            <div class="card-icon email-icon">
              <ion-icon :icon="mailOutline"></ion-icon>
            </div>
            <div class="card-content">
              <span class="card-label">อีเมล</span>
              <span class="card-value">{{ user.email }}</span>
            </div>
          </div>

          <div class="info-card" v-if="user.phoneNumber">
            <div class="card-icon phone-icon">
              <ion-icon :icon="callOutline"></ion-icon>
            </div>
            <div class="card-content">
              <span class="card-label">เบอร์โทร</span>
              <span class="card-value">{{ user.phoneNumber }}</span>
            </div>
          </div>

          <div class="info-card">
            <div class="card-icon uid-icon">
              <ion-icon :icon="fingerPrintOutline"></ion-icon>
            </div>
            <div class="card-content">
              <span class="card-label">User ID</span>
              <span class="card-value uid-text">{{ user.uid }}</span>
            </div>
          </div>
        </div>

        <!-- Action Buttons -->
        <div class="action-section">
          <ion-button expand="block" color="danger" @click="handleLogout">
            <ion-icon slot="start" :icon="logOutOutline"></ion-icon>
            ออกจากระบบ
          </ion-button>
        </div>
      </div>

      <!-- Loading State -->
      <div v-else class="loading-container">
        <ion-spinner name="crescent" color="primary"></ion-spinner>
        <p class="loading-text">กำลังโหลดข้อมูล...</p>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonButtons,
  IonIcon,
  IonSpinner,
  alertController,
  toastController
} from '@ionic/vue';
import {
  personCircle,
  mailOutline,
  callOutline,
  fingerPrintOutline,
  logOutOutline
} from 'ionicons/icons';
import { authService } from '@/auth/auth-service';
import type { AuthUser } from '@/auth/auth-interface';

const router = useRouter();
const user = ref<AuthUser | null>(null);

onMounted(async () => {
  await loadUser();
});

const loadUser = async () => {
  try {
    user.value = await authService.getCurrentUser();
  } catch (error) {
    console.error('Error loading user:', error);
  }
};

const handleLogout = async () => {
  const alert = await alertController.create({
    header: 'ยืนยันการออกจากระบบ',
    message: 'คุณต้องการออกจากระบบใช่หรือไม่?',
    buttons: [
      {
        text: 'ยกเลิก',
        role: 'cancel'
      },
      {
        text: 'ออกจากระบบ',
        role: 'confirm',
        handler: async () => {
          try {
            await authService.logout();
            const toast = await toastController.create({
              message: 'ออกจากระบบสำเร็จ',
              duration: 2000,
              position: 'top',
              color: 'success'
            });
            await toast.present();
            router.push('/login');
          } catch (error: any) {
            const toast = await toastController.create({
              message: error.message || 'เกิดข้อผิดพลาด',
              duration: 2000,
              position: 'top',
              color: 'danger'
            });
            await toast.present();
          }
        }
      }
    ]
  });

  await alert.present();
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&display=swap');

* {
  font-family: 'Kanit', sans-serif;
}

ion-toolbar {
  --background: linear-gradient(135deg, #5eb298 0%, #6e7aaa 100%);
  --color: white;
}

.user-content {
  --background: #f5f5f5;
}

.user-container {
  min-height: 100%;
  padding: 2rem 1.5rem;
}

/* Profile Header */
.profile-header {
  text-align: center;
  margin-bottom: 2rem;
  padding: 2rem 1rem;
  background: linear-gradient(135deg, #5eb298 0%, #6e7aaa 100%);
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(94, 178, 152, 0.3);
}

.profile-avatar {
  width: 120px;
  height: 120px;
  margin: 0 auto 1rem;
  border-radius: 50%;
  overflow: hidden;
  background: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  border: 4px solid rgba(255, 255, 255, 0.3);
}

.profile-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.default-avatar {
  font-size: 5rem;
  color: #5eb298;
}

.profile-name {
  font-size: 1.75rem;
  font-weight: 600;
  color: white;
  margin: 0 0 0.5rem 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.profile-subtitle {
  font-size: 1rem;
  font-weight: 300;
  color: rgba(255, 255, 255, 0.9);
  margin: 0;
}

/* Info Cards */
.info-cards {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 2rem;
}

.info-card {
  background: white;
  border-radius: 16px;
  padding: 1.25rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.info-card:active {
  transform: scale(0.98);
}

.card-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  flex-shrink: 0;
}

.email-icon {
  background: rgba(228, 108, 50, 0.1);
  color: #e46c32;
}

.phone-icon {
  background: rgba(94, 178, 152, 0.1);
  color: #5eb298;
}

.uid-icon {
  background: rgba(110, 122, 170, 0.1);
  color: #6e7aaa;
}

.card-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  overflow: hidden;
}

.card-label {
  font-size: 0.85rem;
  font-weight: 400;
  color: #999;
}

.card-value {
  font-size: 1rem;
  font-weight: 500;
  color: #333;
  word-break: break-word;
}

.uid-text {
  font-size: 0.85rem;
  font-family: 'Courier New', monospace;
  color: #666;
}

/* Action Section */
.action-section {
  margin-top: 2rem;
}

ion-button {
  --border-radius: 12px;
  font-weight: 500;
  height: 48px;
  font-family: 'Kanit', sans-serif;
}

/* Loading State */
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100%;
  gap: 1rem;
}

.loading-text {
  font-size: 1rem;
  font-weight: 400;
  color: #666;
  margin: 0;
}

/* Responsive Design */
@media (min-width: 768px) {
  .user-container {
    max-width: 600px;
    margin: 0 auto;
  }
}
</style>