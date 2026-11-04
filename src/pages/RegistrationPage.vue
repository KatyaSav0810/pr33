<template>
  <div class="registration-page">
    <h1>Регистрация</h1>
    <form @submit.prevent="submitForm" novalidate>
      <div class="form-group">
        <label for="username">Имя пользователя:</label>
        <input
          id="username"
          v-model="form.username"
          type="text"
          @blur="validateField('username')"
          @input="validateField('username')"
          :class="{ error: errors.username }"
        />
        <span v-if="errors.username" class="error-text">{{ errors.username }}</span>
      </div>

      <div class="form-group">
        <label for="email">Email:</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          @blur="validateField('email')"
          @input="validateField('email')"
          :class="{ error: errors.email }"
        />
        <span v-if="errors.email" class="error-text">{{ errors.email }}</span>
      </div>

      <div class="form-group">
        <label for="password">Пароль:</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          @blur="validateField('password')"
          @input="validateField('password')"
          :class="{ error: errors.password }"
        />
        <span v-if="errors.password" class="error-text">{{ errors.password }}</span>
      </div>

      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля:</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          @blur="validateField('confirmPassword')"
          @input="validateField('confirmPassword')"
          :class="{ error: errors.confirmPassword }"
        />
        <span v-if="errors.confirmPassword" class="error-text">{{ errors.confirmPassword }}</span>
      </div>

      <button type="submit" :disabled="!isFormValid">Зарегистрироваться</button>

      <p v-if="successMessage" class="success-text">{{ successMessage }}</p>
      <p v-if="networkError" class="error-text network">{{ networkError }}</p>
      <p v-if="serverError" class="error-text server">{{ serverError }}</p>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';

const form = ref({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

const errors = ref({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

const successMessage = ref('');
const networkError = ref('');
const serverError = ref('');

// Правила проверки пароля (п.7 методички)
const passwordRules = {
  minLength: (value) => value.length >= 8,
  hasLowercase: (value) => /[a-z]/.test(value),
  hasUppercase: (value) => /[A-Z]/.test(value),
  hasDigit: (value) => /[0-9]/.test(value),
  hasSpecial: (value) => /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(value)
};

// Живая валидация (п.8 методички): @blur и @input
const validateField = (field) => {
  errors.value[field] = '';

  switch (field) {
    case 'username':
      if (!form.value.username.trim()) {
        errors.value.username = 'Имя пользователя обязательно';
      } else if (form.value.username.length < 3) {
        errors.value.username = 'Минимум 3 символа';
      }
      break;

    case 'email':
      if (!form.value.email.trim()) {
        errors.value.email = 'Email обязателен';
      } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email)) {
        errors.value.email = 'Некорректный email';
      }
      break;

    case 'password':
      if (!form.value.password) {
        errors.value.password = 'Пароль обязателен';
      } else {
        const pwd = form.value.password;
        if (!passwordRules.minLength(pwd)) {
          errors.value.password = 'Минимум 8 символов';
        } else if (!passwordRules.hasLowercase(pwd)) {
          errors.value.password = 'Нужна строчная буква';
        } else if (!passwordRules.hasUppercase(pwd)) {
          errors.value.password = 'Нужна прописная буква';
        } else if (!passwordRules.hasDigit(pwd)) {
          errors.value.password = 'Нужна цифра';
        } else if (!passwordRules.hasSpecial(pwd)) {
          errors.value.password = 'Нужен специальный символ';
        }
      }
      break;

    case 'confirmPassword':
      if (!form.value.confirmPassword) {
        errors.value.confirmPassword = 'Подтвердите пароль';
      } else if (form.value.password !== form.value.confirmPassword) {
        errors.value.confirmPassword = 'Пароли не совпадают';
      }
      break;
  }
};

// Кнопка активна только когда форма валидна
const isFormValid = computed(() => {
  return (
    form.value.username.trim().length >= 3 &&
    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email) &&
    passwordRules.minLength(form.value.password) &&
    passwordRules.hasLowercase(form.value.password) &&
    passwordRules.hasUppercase(form.value.password) &&
    passwordRules.hasDigit(form.value.password) &&
    passwordRules.hasSpecial(form.value.password) &&
    form.value.password === form.value.confirmPassword
  );
});

// Отправка через axios (п.9 методички)
const submitForm = async () => {
  successMessage.value = '';
  networkError.value = '';
  serverError.value = '';

  ['username', 'email', 'password', 'confirmPassword'].forEach(validateField);

  if (!isFormValid.value) return;

  try {
    const response = await axios.post('https://jsonplaceholder.typicode.com/users', {
      name: form.value.username,
      email: form.value.email
    });

    successMessage.value = `Пользователь ${response.data.name} успешно зарегистрирован!`;
    form.value = { username: '', email: '', password: '', confirmPassword: '' };
  } catch (error) {
    // Разные сообщения для сети и сервера (п.9 методички)
    if (error.response) {
      serverError.value = `Ошибка сервера: ${error.response.status}. Попробуйте позже.`;
    } else if (error.request) {
      networkError.value = 'Ошибка сети. Проверьте подключение к интернету.';
    } else {
      serverError.value = `Ошибка: ${error.message}`;
    }
  }
};
</script>
<style scoped>
.registration-page {
  max-width: 400px;
  margin: 2rem auto;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.3rem;
}

input {
  width: 100%;
  padding: 0.4rem;
  border: 1px solid #ccc;
  border-radius: 3px;
  box-sizing: border-box;
}

input.error {
  border-color: red;
}

.error-text {
  color: red;
  font-size: 0.85rem;
  display: block;
  margin-top: 0.2rem;
}

.success-text {
  color: green;
  font-weight: 600;
  margin-top: 1rem;
}

button {
  width: 100%;
  padding: 0.6rem;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

button:disabled {
  background: #aaa;
  cursor: not-allowed;
}
</style>