
<script setup>
import { ref, watch } from 'vue'

const props = defineProps({ visible: Boolean })
const emit = defineEmits(['update:visible', 'success', 'cancel'])

const cardNumber = ref('')
const cardName = ref('')
const exp = ref('')
const cvv = ref('')
const error = ref('')
const loading = ref(false)

function close() { emit('update:visible', false) }
function cancel() { error.value = ''; emit('cancel'); emit('update:visible', false) }

function formatNumber(e) {
  cardNumber.value = e.target.value.replace(/\D/g, '').slice(0,16).replace(/(.{4})/g, '$1 ').trim()
}

function luhnCheck(num) {
  const digits = num.replace(/\D/g,'').split('').reverse().map(d=>+d)
  let sum = 0
  for (let i=0;i<digits.length;i++){
    let d = digits[i]
    if (i % 2 === 1) {
      d *= 2
      if (d > 9) d -= 9
    }
    sum += d
  }
  return sum % 10 === 0
}

async function confirmPayment() {
  error.value = ''
  const raw = cardNumber.value.replace(/\s/g,'')
  if (!raw || raw.length < 12) { error.value = 'Введіть правильний номер картки'; return }
  if (!/^\d{2}\/\d{2}$/.test(exp.value)) { error.value = 'Введіть термін у MM/YY'; return }
  if (!/^\d{3,4}$/.test(cvv.value)) { error.value = 'Введіть CVV (3-4 цифри)'; return }

  const isTestCard = raw === '4242424242424242'
  if (!isTestCard && !luhnCheck(raw)) {
    error.value = 'Номер картки не пройшов перевірку (тільки тестові картки для демо)'
    return
  }

  loading.value = true
  await new Promise(r => setTimeout(r, 900))

  emit('success', { cardLast4: raw.slice(-4), cardName: cardName.value })
  emit('update:visible', false)
  loading.value = false
}
</script>



<template>
  <transition name="fade">
    <div v-if="visible" class="modal">
      <div class="modal-content payment-modal">
        <button class="close-btn" @click="close">&times;</button>
        <h2>Оплата (демо)</h2>

        <div class="field">
          <label>Ім'я на карті</label>
          <input v-model="cardName" placeholder="Ivan Ivanov" />
        </div>

        <div class="field">
          <label>Номер картки</label>
          <input v-model="cardNumber" placeholder="4242 4242 4242 4242" maxlength="19" @input="formatNumber" />
        </div>

        <div class="row">
          <div class="field">
            <label>Термін (MM/YY)</label>
            <input v-model="exp" placeholder="12/25" maxlength="5" />
          </div>
          <div class="field">
            <label>CVV</label>
            <input v-model="cvv" placeholder="123" maxlength="4" />
          </div>
        </div>

        <p v-if="error" class="error">{{ error }}</p>

        <div class="actions">
          <button @click="cancel">Скасування</button>
          <button :disabled="loading" @click="confirmPayment">
            {{ loading ? 'Обробка...' : 'Сплатити' }}
          </button>
        </div>

        <p class="hint">Для демо використовуйте: <b>4242 4242 4242 4242</b> (поверне успіх)</p>
      </div>
    </div>
  </transition>
</template>


<style scoped>

.modal {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(3px);
  z-index: 1000;
  animation: fadeIn 0.25s ease;
}

.payment-modal {
  background: var(--light);
  border: 2px solid var(--highlight);
  padding: 24px 28px;
  border-radius: 16px;
  width: min(420px, 90vw);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.25);
  position: relative;
  color: var(--accent);
  animation: slideIn 0.3s ease;
}

.row .field {
  flex: 1;
  min-width: 0; /* важно! предотвращает переполнение */
}

.payment-modal h2 {
  margin-bottom: 14px;
  color: var(--accent-heading);
  text-align: center;
  font-size: 1.4em;
}

.row {
  display: flex;
  gap: 10px;
}

.field {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
}

.field label {
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 4px;
  color: var(--accent);
}

.field input {
  padding: 8px 10px;
  border: 1px solid var(--highlight);
  border-radius: 6px;
  font-size: 14px;
  background: var(--background);
  color: var(--accent);
  transition: border-color 0.2s, background-color 0.2s;
}

.field input:focus {
  outline: none;
  border-color: var(--secondary);
  background-color: var(--light);
}

.actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 10px;
}

.actions button {
  padding: 8px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: background 0.25s, transform 0.1s;
}

.actions button:first-child {
  background: var(--highlight);
  color: var(--accent);
}

.actions button:last-child {
  background: var(--accent);
  color: var(--light);
}

.actions button:hover {
  background: var(--secondary);
  color: var(--light);
  transform: translateY(-1px);
}

.error {
  color: #c33;
  font-size: 13px;
  margin-top: 4px;
}

.hint {
  font-size: 12px;
  color: var(--accent);
  margin-top: 8px;
  text-align: center;
}

.close-btn {
  position: absolute;
  right: 12px;
  top: 8px;
  background: none;
  border: none;
  font-size: 22px;
  cursor: pointer;
  color: var(--accent);
  transition: color 0.2s;
}

.close-btn:hover {
  color: var(--secondary);
}

/* 🔹 Анимации */
@keyframes fadeIn {
  from { opacity: 0 }
  to { opacity: 1 }
}

@keyframes slideIn {
  from { transform: translateY(-20px); opacity: 0 }
  to { transform: translateY(0); opacity: 1 }
}
</style>
