<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>รายการรายรับ-รายจ่าย</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="goToAdd">
            <ion-icon slot="icon-only" :icon="addCircleOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- สรุปยอด -->
      <ion-card>
        <ion-card-content>
          <ion-grid>
            <ion-row>
              <ion-col size="6">
                <div class="summary-item income">
                  <ion-icon :icon="arrowUpCircleOutline"></ion-icon>
                  <div class="summary-label">รายรับ</div>
                  <div class="summary-amount">{{ formatCurrency(totalIncome) }}</div>
                </div>
              </ion-col>
              <ion-col size="6">
                <div class="summary-item expense">
                  <ion-icon :icon="arrowDownCircleOutline"></ion-icon>
                  <div class="summary-label">รายจ่าย</div>
                  <div class="summary-amount">{{ formatCurrency(totalExpense) }}</div>
                </div>
              </ion-col>
            </ion-row>
            <ion-row>
              <ion-col>
                <div class="summary-item balance" :class="{ negative: balance < 0 }">
                  <div class="summary-label">คงเหลือ</div>
                  <div class="summary-amount">{{ formatCurrency(balance) }}</div>
                </div>
              </ion-col>
            </ion-row>
          </ion-grid>
        </ion-card-content>
      </ion-card>

      <!-- รายการ -->
      <ion-list v-if="expenses.length > 0">
        <ion-item 
          v-for="expense in expenses" 
          :key="expense.id"
          @click="goToEdit(expense.id)"
          button>
          <ion-icon 
            slot="start" 
            :icon="expense.type === 'income' ? arrowUpCircle : arrowDownCircle"
            :color="expense.type === 'income' ? 'success' : 'danger'">
          </ion-icon>
          
          <ion-label>
            <h2>{{ expense.title }}</h2>
            <p>{{ expense.category }} • {{ formatDate(expense.createdAt) }}</p>
            <p v-if="expense.note" class="note">{{ expense.note }}</p>
          </ion-label>

          <ion-note 
            slot="end" 
            :color="expense.type === 'income' ? 'success' : 'danger'">
            {{ expense.type === 'income' ? '+' : '-' }}{{ formatCurrency(expense.amount) }}
          </ion-note>
        </ion-item>
      </ion-list>

      <!-- ไม่มีข้อมูล -->
      <div v-else class="empty-state">
        <ion-icon :icon="walletOutline" size="large"></ion-icon>
        <p>ยังไม่มีรายการ</p>
        <ion-button @click="goToAdd">เพิ่มรายการแรก</ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonList, IonItem, IonLabel, IonNote, IonIcon, IonButton,
  IonButtons, IonCard, IonCardContent, IonGrid, IonRow, IonCol,
  useIonRouter
} from "@ionic/vue";
import {
  addCircleOutline,
  arrowUpCircleOutline,
  arrowDownCircleOutline,
  arrowUpCircle,
  arrowDownCircle,
  walletOutline
} from "ionicons/icons";


import { database } from "@/firebase";
import {
  ref as dbRef,
  onValue,
  off,
  query,
  orderByChild
} from "firebase/database";

const formatDate = (timestamp: number) => {
  if (!timestamp) return '';

  const d = new Date(timestamp);

  return d.toLocaleDateString('th-TH', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  });
};

const router = useIonRouter();

interface Expense {
  id: string;
  title: string;
  amount: number;
  type: string;
  category: string;
  note: string;
  createdAt: number; 
}


const expenses = ref<Expense[]>([]);

// คำนวณยอดรวม
const totalIncome = computed(() => {
  return expenses.value
    .filter(e => e.type === 'income')
    .reduce((sum, e) => sum + e.amount, 0);
});

const totalExpense = computed(() => {
  return expenses.value
    .filter(e => e.type === 'expense')
    .reduce((sum, e) => sum + e.amount, 0);
});

const balance = computed(() => {
  return totalIncome.value - totalExpense.value;
});

// ฟังก์ชันช่วย
const formatCurrency = (amount: number) => {
  return amount.toLocaleString('th-TH', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }) + ' ฿';
};

const goToAdd = () => {
  router.push('/add-expense');
};

const goToEdit = (id: string) => {
  router.push(`/edit-expense/${id}`);
};

// ดึงข้อมูลแบบ Realtime
let expensesRef: any = null;

onMounted(() => {
  expensesRef = query(
    dbRef(database, "expenses"),
    orderByChild("createdAt")
  );

  onValue(expensesRef, (snapshot) => {
    const data = snapshot.val();

    if (!data) {
      expenses.value = [];
      return;
    }

    expenses.value = Object.entries(data)
      .map(([id, value]: any) => ({
        id,
        ...value
      }))
      .sort((a, b) => b.createdAt - a.createdAt); // ใหม่ → เก่าสุดล่าง
  });
});

onUnmounted(() => {
  if (expensesRef) {
    off(expensesRef);
  }
});

</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&display=swap');

* {
  font-family: 'Kanit', sans-serif;
}

ion-header ion-toolbar {
  --background: linear-gradient(135deg, #e46c32 0%, #5eb298 100%);
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

/* Card สรุปยอด */
ion-card {
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(228, 108, 50, 0.15);
  margin: 16px;
  background: white;
  border-top: 4px solid #e46c32;
}

.summary-item {
  text-align: center;
  padding: 20px 15px;
  border-radius: 12px;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.summary-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, transparent 0%, rgba(255, 255, 255, 0.3) 100%);
  pointer-events: none;
}

.summary-item.income {
  background: linear-gradient(135deg, #5eb298 0%, #4da085 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(94, 178, 152, 0.3);
}

.summary-item.expense {
  background: linear-gradient(135deg, #e46c32 0%, #d45a28 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(228, 108, 50, 0.3);
}

.summary-item.balance {
  background: linear-gradient(135deg, #6e7aaa 0%, #5c6890 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(110, 122, 170, 0.3);
}

.summary-item.balance.negative {
  background: linear-gradient(135deg, #b24342 0%, #9d3635 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(178, 67, 66, 0.3);
}

.summary-item ion-icon {
  font-size: 32px;
  margin-bottom: 8px;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
}

.summary-label {
  font-size: 0.95rem;
  margin-bottom: 8px;
  font-weight: 500;
  opacity: 0.95;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.summary-amount {
  font-size: 1.5rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* รายการ */
ion-list {
  background: transparent;
  padding: 0 16px 16px 16px;
}

ion-item {
  --background: white;
  --border-radius: 12px;
  margin-bottom: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  border-left: 4px solid #e46c32;
  transition: all 0.3s ease;
  overflow: hidden;
}

ion-item:hover {
  box-shadow: 0 4px 12px rgba(228, 108, 50, 0.2);
  transform: translateX(4px);
}

ion-item::part(native) {
  padding: 16px;
}

ion-label h2 {
  font-weight: 600;
  font-size: 1.1rem;
  color: #5f4256;
  margin-bottom: 4px;
}

ion-label p {
  font-size: 0.9rem;
  color: #6e7aaa;
  font-weight: 400;
}

ion-label p.note {
  font-size: 0.85rem;
  color: #999;
  font-style: italic;
  margin-top: 4px;
}

ion-note {
  font-size: 1.1rem;
  font-weight: 600;
  padding: 8px 12px;
  border-radius: 8px;
  background: rgba(228, 108, 50, 0.1);
}

ion-note[color="success"] {
  background: rgba(94, 178, 152, 0.15);
  color: #5eb298;
}

ion-note[color="danger"] {
  background: rgba(228, 108, 50, 0.15);
  color: #e46c32;
}

/* ไม่มีข้อมูล */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60vh;
  color: #6e7aaa;
}

.empty-state ion-icon {
  font-size: 100px;
  margin-bottom: 24px;
  color: #e46c32;
  opacity: 0.5;
}

.empty-state p {
  font-size: 1.2rem;
  font-weight: 500;
  margin-bottom: 24px;
  color: #5f4256;
}

.empty-state ion-button {
  --background: linear-gradient(135deg, #e46c32 0%, #5eb298 100%);
  --border-radius: 12px;
  --box-shadow: 0 4px 12px rgba(228, 108, 50, 0.3);
  font-weight: 600;
  letter-spacing: 0.5px;
  text-transform: none;
}

/* ปุ่มเพิ่ม */
ion-header ion-button {
  --color: white;
}

ion-header ion-button ion-icon {
  font-size: 28px;
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

ion-card {
  animation: fadeInUp 0.5s ease-out;
}

ion-item {
  animation: fadeInUp 0.4s ease-out;
}

ion-item:nth-child(1) { animation-delay: 0.1s; }
ion-item:nth-child(2) { animation-delay: 0.15s; }
ion-item:nth-child(3) { animation-delay: 0.2s; }
ion-item:nth-child(4) { animation-delay: 0.25s; }
ion-item:nth-child(5) { animation-delay: 0.3s; }

/* Responsive */
@media (min-width: 768px) {
  ion-card {
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
  }
  
  ion-list {
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
  }
}
</style>