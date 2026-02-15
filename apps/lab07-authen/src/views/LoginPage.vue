<template>
  <ion-page>
    <ion-content :fullscreen="true" class="login-content">
      <div class="login-container">
        <!-- Logo/Brand Section -->
        <div class="brand-section">
          <div class="logo-circle">
            <ion-icon :icon="personCircleOutline" class="logo-icon"></ion-icon>
          </div>
          <h1 class="brand-title">ยินดีต้อนรับ</h1>
          <p class="brand-subtitle">เข้าสู่ระบบเพื่อดำเนินการต่อ</p>
        </div>

        <!-- Login Methods -->
        <div class="login-methods">
          <!-- Email/Password Login -->
          <button class="login-btn email-btn" @click="loginWithEmail">
            <div class="btn-icon">
              <ion-icon :icon="mailOutline"></ion-icon>
            </div>
            <div class="btn-content">
              <span class="btn-title">เข้าสู่ระบบด้วยอีเมล</span>
              <span class="btn-subtitle">ใช้อีเมลและรหัสผ่าน</span>
            </div>
            <ion-icon :icon="chevronForwardOutline" class="btn-arrow"></ion-icon>
          </button>

          <!-- Google Login -->
          <button class="login-btn google-btn" @click="loginWithGoogle">
            <div class="btn-icon">
              <ion-icon :icon="logoGoogle"></ion-icon>
            </div>
            <div class="btn-content">
              <span class="btn-title">เข้าสู่ระบบด้วย Google</span>
              <span class="btn-subtitle">ใช้บัญชี Google ของคุณ</span>
            </div>
            <ion-icon :icon="chevronForwardOutline" class="btn-arrow"></ion-icon>
          </button>

          <!-- Phone Login -->
          <button class="login-btn phone-btn" @click="loginWithPhone">
            <div class="btn-icon">
              <ion-icon :icon="callOutline"></ion-icon>
            </div>
            <div class="btn-content">
              <span class="btn-title">เข้าสู่ระบบด้วยเบอร์โทร</span>
              <span class="btn-subtitle">รับรหัส OTP ทาง SMS</span>
            </div>
            <ion-icon :icon="chevronForwardOutline" class="btn-arrow"></ion-icon>
          </button>
        </div>

        <!-- Footer -->
        <div class="login-footer">
          <p class="footer-text">
            การเข้าสู่ระบบแสดงว่าคุณยอมรับ
            <a href="#" class="footer-link">ข้อกำหนดการใช้งาน</a>
            และ
            <a href="#" class="footer-link">นโยบายความเป็นส่วนตัว</a>
          </p>
        </div>

        <!-- reCAPTCHA Container for Web Phone Auth -->
        <div id="recaptcha-container"></div>
      </div>

      <!-- Loading Overlay -->
      <div v-if="isLoading" class="loading-overlay">
        <ion-spinner name="crescent" color="light"></ion-spinner>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonPage,
  IonContent,
  IonIcon,
  IonSpinner,
  toastController,
  alertController,
  modalController
} from '@ionic/vue';
import {
  personCircleOutline,
  mailOutline,
  logoGoogle,
  callOutline,
  chevronForwardOutline
} from 'ionicons/icons';
import { authService } from '@/auth/auth-service';
import type { EmailPasswordCredentials, PhoneCredentials } from '@/auth/auth-interface';

const router = useRouter();
const isLoading = ref(false);

const showToast = async (message: string, color: string = 'danger') => {
  const toast = await toastController.create({
    message,
    duration: 2000,
    position: 'top',
    color
  });
  await toast.present();
};

const loginWithEmail = async () => {
  const alert = await alertController.create({
    header: 'เข้าสู่ระบบด้วยอีเมล',
    inputs: [
      {
        name: 'email',
        type: 'email',
        placeholder: 'อีเมล',
      },
      {
        name: 'password',
        type: 'password',
        placeholder: 'รหัสผ่าน',
      },
    ],
    buttons: [
      {
        text: 'ยกเลิก',
        role: 'cancel',
      },
      {
        text: 'เข้าสู่ระบบ',
        handler: async (data) => {
          if (!data.email || !data.password) {
            await showToast('กรุณากรอกอีเมลและรหัสผ่าน');
            return false;
          }

          try {
            isLoading.value = true;
            const credentials: EmailPasswordCredentials = {
              email: data.email,
              password: data.password
            };
            await authService.loginWithEmailPassword(credentials);
            await showToast('เข้าสู่ระบบสำเร็จ', 'success');
            router.push('/tabs/tab1');
          } catch (error: any) {
            await showToast(error.message || 'เข้าสู่ระบบไม่สำเร็จ');
          } finally {
            isLoading.value = false;
          }
        },
      },
    ],
  });

  await alert.present();
};

const loginWithGoogle = async () => {
  try {
    isLoading.value = true;
    await authService.loginWithGoogle();
    await showToast('เข้าสู่ระบบสำเร็จ', 'success');
    router.push('/tabs/tab1');
  } catch (error: any) {
    await showToast(error.message || 'เข้าสู่ระบบไม่สำเร็จ');
  } finally {
    isLoading.value = false;
  }
};

const loginWithPhone = async () => {
  // Step 1: Get phone number
  const phoneAlert = await alertController.create({
    header: 'เข้าสู่ระบบด้วยเบอร์โทร',
    message: 'กรุณากรอกเบอร์โทรศัพท์ในรูปแบบ +66812345678',
    inputs: [
      {
        name: 'phoneNumber',
        type: 'tel',
        placeholder: '+66812345678',
        value: '+66'
      },
    ],
    buttons: [
      {
        text: 'ยกเลิก',
        role: 'cancel',
      },
      {
        text: 'ส่ง OTP',
        handler: async (data) => {
          if (!data.phoneNumber) {
            await showToast('กรุณากรอกเบอร์โทรศัพท์');
            return false;
          }

          try {
            isLoading.value = true;
            const phoneCredentials: PhoneCredentials = {
              phoneNumberE164: data.phoneNumber
            };
            
            const { verificationId } = await authService.startPhoneLogin(phoneCredentials);
            isLoading.value = false;

            // Step 2: Get OTP code
            const otpAlert = await alertController.create({
              header: 'ยืนยัน OTP',
              message: `รหัส OTP ถูกส่งไปที่ ${data.phoneNumber}`,
              inputs: [
                {
                  name: 'code',
                  type: 'text',
                  placeholder: 'รหัส OTP 6 หลัก',
                  attributes: {
                    maxlength: 6,
                  }
                },
              ],
              buttons: [
                {
                  text: 'ยกเลิก',
                  role: 'cancel',
                },
                {
                  text: 'ยืนยัน',
                  handler: async (otpData) => {
                    if (!otpData.code) {
                      await showToast('กรุณากรอกรหัส OTP');
                      return false;
                    }

                    try {
                      isLoading.value = true;
                      await authService.confirmPhoneCode({
                        verificationId,
                        verificationCode: otpData.code
                      });
                      await showToast('เข้าสู่ระบบสำเร็จ', 'success');
                      router.push('/tabs/tab1');
                    } catch (error: any) {
                      await showToast(error.message || 'รหัส OTP ไม่ถูกต้อง');
                    } finally {
                      isLoading.value = false;
                    }
                  },
                },
              ],
            });

            await otpAlert.present();

          } catch (error: any) {
            isLoading.value = false;
            await showToast(error.message || 'ส่ง OTP ไม่สำเร็จ');
          }
        },
      },
    ],
  });

  await phoneAlert.present();
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&display=swap');

* {
  font-family: 'Kanit', sans-serif;
}

.login-content {
  --background: linear-gradient(135deg, #5eb298 0%, #6e7aaa 100%);
}

.login-container {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 2rem 1.5rem;
  position: relative;
}

/* reCAPTCHA Container - Hidden */
#recaptcha-container {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

/* Brand Section */
.brand-section {
  text-align: center;
  margin-bottom: 3rem;
}

.logo-circle {
  width: 100px;
  height: 100px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1.5rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.logo-icon {
  font-size: 3.5rem;
  color: white;
}

.brand-title {
  font-size: 2rem;
  font-weight: 600;
  color: white;
  margin: 0 0 0.5rem 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.brand-subtitle {
  font-size: 1rem;
  font-weight: 300;
  color: rgba(255, 255, 255, 0.9);
  margin: 0;
}

/* Login Methods */
.login-methods {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 2rem;
}

.login-btn {
  background: white;
  border: none;
  border-radius: 16px;
  padding: 1.25rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.login-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, transparent 0%, rgba(0, 0, 0, 0.05) 100%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.login-btn:hover::before {
  opacity: 1;
}

.login-btn:active {
  transform: scale(0.98);
}

.btn-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  font-size: 1.5rem;
}

.email-btn .btn-icon {
  background: rgba(228, 108, 50, 0.1);
  color: #e46c32;
}

.google-btn .btn-icon {
  background: rgba(94, 178, 152, 0.1);
  color: #5eb298;
}

.phone-btn .btn-icon {
  background: rgba(110, 122, 170, 0.1);
  color: #6e7aaa;
}

.btn-content {
  flex: 1;
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.btn-title {
  font-size: 1rem;
  font-weight: 500;
  color: #333;
  display: block;
}

.btn-subtitle {
  font-size: 0.85rem;
  font-weight: 300;
  color: #666;
  display: block;
}

.btn-arrow {
  font-size: 1.25rem;
  color: #999;
  flex-shrink: 0;
  transition: transform 0.3s ease;
}

.login-btn:hover .btn-arrow {
  transform: translateX(4px);
}

/* Footer */
.login-footer {
  text-align: center;
  margin-top: auto;
  padding-top: 2rem;
}

.footer-text {
  font-size: 0.85rem;
  font-weight: 300;
  color: rgba(255, 255, 255, 0.9);
  line-height: 1.6;
  margin: 0;
}

.footer-link {
  color: white;
  font-weight: 500;
  text-decoration: none;
  border-bottom: 1px solid rgba(255, 255, 255, 0.5);
  transition: border-color 0.3s ease;
}

.footer-link:hover {
  border-bottom-color: white;
}

/* Loading Overlay */
.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.loading-overlay ion-spinner {
  width: 48px;
  height: 48px;
}

/* Responsive Design */
@media (min-width: 768px) {
  .login-container {
    max-width: 480px;
    margin: 0 auto;
  }
}

@media (max-height: 700px) {
  .brand-section {
    margin-bottom: 2rem;
  }

  .logo-circle {
    width: 80px;
    height: 80px;
    margin-bottom: 1rem;
  }

  .logo-icon {
    font-size: 3rem;
  }

  .brand-title {
    font-size: 1.75rem;
  }
}
</style>