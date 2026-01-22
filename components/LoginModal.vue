<script setup>
import { ref, watch } from 'vue'
import { useToast } from 'vue-toastification'

const props = defineProps({
  visible: Boolean
})

const emit = defineEmits(['update:visible'])

const toast = useToast()

const email = ref('')
const code = ref('')
const step = ref(1)
const isEmailValid = ref(false)

function validateEmail() {
  isEmailValid.value = /\S+@\S+\.\S+/.test(email.value)
}

function close() {
  emit('update:visible', false)
}

async function sendCode() {
  try {
    const res = await fetch('http://localhost:3000/api/send-code', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value })
    })
    const data = await res.json()
    if (data.success) {
      toast.success('✅ Код відправлено на пошту!')
      step.value = 2
    } else {
      toast.error('❌ Не вдалося відправити код')
    }
  } catch {
    toast.error('⚠️ Помилка сервера')
  }
}

async function verifyCode() {
  try {
    const res = await fetch('http://localhost:3000/api/verify-code', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value, code: code.value })
    })

    const data = await res.json()

    if (data.success) {
      localStorage.setItem('userId', data.userId)
      localStorage.setItem('role', data.role)
      localStorage.setItem('loginTime', Date.now())

      toast.success('✅ Вхід виконано успішно!')

      emit('update:visible', false)

      emit('logged-in')
    } else {
      toast.error('❌ Невірний код')
    }
  } catch (err) {
    console.error('Помилка при перевірці коду:', err)
    toast.error('⚠️ Помилка сервера')
  }
}
</script>


<template>
  <transition name="fade">
    <div v-if="visible" class="modal">
      <div class="modal-content login-modal">
        <span class="close-btn" @click="close">&times;</span>

        <!-- ВВОД ПОЧТЫ -->
        <div v-if="step === 1">
          <h2 class="login-title">{{ $t('login_email.title') }}</h2>
          <p class="login-subtitle">{{ $t('login_email.subtitle') }}</p>

          <div class="field login-field">
            <div class="label">Email</div>
            <input
              class="input"
              type="email"
              v-model="email"
              placeholder="example@gmail.com"
              @input="validateEmail"
            />
          </div>

          <button
            class="search-btn continue-btn"
            :disabled="!isEmailValid"
            @click="sendCode"
          >
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" viewBox="0 0 16 16" class="arrow-icon"> <path fill-rule="evenodd" d="M1 8a.5.5 0 0 1 .5-.5h11.793l-3.147-3.146a.5.5 0 1 1 .708-.708l4 4a.5.5 0 0 1 0 .708l-4 4a.5.5 0 0 1-.708-.708L13.293 8.5H1.5A.5.5 0 0 1 1 8z"/> </svg>
            Continue
          </button>
        </div>

        <!-- ВВОД КОДА -->
        <div v-else>
          <h2 class="login-title">Введіть код підтвердження</h2>
          <p class="login-subtitle">
            Ми надіслали код на <b>{{ email }}</b>
          </p>

          <div class="field login-field">
            <div class="label">Код</div>
            <input
              class="input"
              type="text"
              v-model="code"
              maxlength="6"
              placeholder="123456"
            />
          </div>

          <button
            class="search-btn verify-btn"
            :disabled="code.length !== 6"
            @click="verifyCode"
          >
            Перевірити
          </button>

          <button class="back-btn" @click="step = 1">← Назад</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<style scoped>
</style>
