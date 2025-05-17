
<script setup>

//AuthForm.vue file 
import { ref, defineEmits } from 'vue'

const emit = defineEmits(['authenticated'])

const auth = ref({
  mode: 'login',
  credentials: {
    username: '',
    password: ''
  },
  error: ''
})

const handleAuth = () => {
  if (!auth.value.credentials.username.trim()) {
    auth.value.error = 'Username is required'
    return
  }

  if (!auth.value.credentials.password) {
    auth.value.error = 'Password is required'
    return
  }

  if (auth.value.mode === 'signup') {
    // In a real app, you would validate password strength here
    console.log('Signing up new user:', auth.value.credentials.username)
  }

  // Clear any previous errors
  auth.value.error = ''
  
  // Emit the authentication event
  emit('authenticated', auth.value.credentials.username)
  
  // Reset form
  auth.value.credentials = { username: '', password: '' }
}

const toggleAuthMode = () => {
  auth.value.mode = auth.value.mode === 'login' ? 'signup' : 'login'
  auth.value.error = ''
}
</script>

<template>
  <div class="auth-screen">
    <div class="auth-box">
      <h2>{{ auth.mode === 'login' ? 'Login' : 'Sign Up' }}</h2>

      <form @submit.prevent="handleAuth">
        <div class="form-group">
          <input 
            type="text" 
            placeholder="Username" 
            v-model="auth.credentials.username" 
            required
          >
        </div>

        <div class="form-group">
          <input 
            type="password" 
            placeholder="Password" 
            v-model="auth.credentials.password" 
            required
          >
        </div>

        <div v-if="auth.error" class="error-message">
          {{ auth.error }}
        </div>

        <button type="submit">
          {{ auth.mode === 'login' ? 'Login' : 'Sign Up' }}
        </button>
      </form>

      <p class="auth-toggle" @click="toggleAuthMode">
        {{ auth.mode === 'login' ? 'Need an account? Sign up' : 'Already have an account? Login' }}
      </p>
    </div>
  </div>
</template>