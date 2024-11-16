<script setup lang="ts">
import { ref } from 'vue'
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  sendEmailVerification,
  onAuthStateChanged,
  GoogleAuthProvider,
  signInWithPopup,
} from 'firebase/auth'
import { auth } from '@/util/firebase'

const provider = new GoogleAuthProvider()

// Interfaces
interface FormData {
  email: string
  password: string
}

interface FormErrors {
  email: string
  password: string
}

interface ValidationRules {
  required: boolean
  minLength?: number
  pattern?: RegExp
}

interface FieldValidationRules {
  [key: string]: ValidationRules
}

// Constants
const EMAIL_PATTERN = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

const validationRules: FieldValidationRules = {
  email: {
    required: true,
    pattern: EMAIL_PATTERN,
  },
  password: {
    required: true,
    minLength: 6,
  },
}

// State
const isLogin = ref<boolean>(true)
const showPassword = ref<boolean>(false)
const formData = ref<FormData>({
  email: '',
  password: '',
})
const errors = ref<FormErrors>({
  email: '',
  password: '',
})

const validateField = (field: keyof FormData, value: string): string => {
  const rules = validationRules[field]
  if (rules.required && !value) {
    return `${field.charAt(0).toUpperCase() + field.slice(1)} is required`
  }

  if (rules.pattern && !rules.pattern.test(value)) {
    return `Please enter a valid ${field}`
  }

  if (rules.minLength && value.length < rules.minLength) {
    return `${field.charAt(0).toUpperCase() + field.slice(1)} must be at least ${rules.minLength} characters`
  }

  return ''
}

const validateForm = (): boolean => {
  let isValid = true

  // Validate each field
  Object.keys(formData.value).forEach(field => {
    const error = validateField(
      field as keyof FormData,
      formData.value[field as keyof FormData],
    )
    if (error) {
      errors[field as keyof FormErrors] = error
      isValid = false
    }
  })

  return isValid
}

const clearError = (field: keyof FormErrors): void => {
  errors[field as keyof FormErrors] = ''
}

const resetForm = (): void => {
  formData.value.email = ''
  formData.value.password = ''
  Object.keys(errors).forEach(field => {
    errors[field as keyof FormErrors] = ''
  })
}

const handleSubmit = async (): Promise<void> => {
  if (!validateForm()) return
  try {
    await submitForm(formData.value, isLogin.value)
  } catch (error) {
    console.error('Error submitting form:', error)
  }
}

const handleGoogleAuth = async (): Promise<void> => {
  try {
    const result = await signInWithPopup(auth, provider)
    // This gives you a Google Access Token. You can use it to access the Google API.
    /*     const credential = GoogleAuthProvider.credentialFromResult(result)
    const token = credential.accessToken */

    // const user = result.user
  } catch (error) {
    console.error('Google auth error:', error)
    /*     // Handle Errors here.
    const errorCode = error.code
    const errorMessage = error.message
    // The email of the user's account used.
    const email = error.customData.email
    // The AuthCredential type that was used.
    const credential = GoogleAuthProvider.credentialFromError(error) */
  }
}

const submitForm = async (data: FormData, isLogin: boolean): Promise<any> => {
  try {
    if (isLogin) {
      const userCredential = await signInWithEmailAndPassword(
        auth,
        data.email,
        data.password,
      )
      console.log('login', userCredential)
      return
    }

    const userCredential = await createUserWithEmailAndPassword(
      auth,
      data.email,
      data.password,
    )
    console.log('sign up', userCredential)
    sendEmailVerification(auth.currentUser)
  } catch (err) {
    console.error(err)
  }
}

const toggleForm = (): void => {
  isLogin.value = !isLogin.value
  resetForm()
}
</script>

<template>
  <div class="auth-container">
    <div class="auth-card">
      <h1 class="auth-title">{{ isLogin ? 'Login' : 'Sign Up' }}</h1>
      <form @submit.prevent="handleSubmit" class="auth-form">
        <div class="form-group">
          <label for="email">Email:</label>
          <input
            type="email"
            id="email"
            v-model="formData.email"
            :class="{ error: errors.email }"
            @focus="clearError('email')"
          />
          <span v-if="errors.email" class="error-message">{{
            errors.email
          }}</span>
        </div>

        <div class="form-group">
          <label for="password">Password:</label>
          <div class="password-input">
            <input
              :type="showPassword ? 'text' : 'password'"
              id="password"
              v-model="formData.password"
              :class="{ error: errors.password }"
              @focus="clearError('password')"
            />
            <button
              type="button"
              class="toggle-password"
              @click="showPassword = !showPassword"
            >
              {{ showPassword ? '👁️' : '👁️‍🗨️' }}
            </button>
          </div>
          <span v-if="errors.password" class="error-message">{{
            errors.password
          }}</span>
        </div>

        <button type="submit" class="submit-btn">
          {{ isLogin ? 'Log in' : 'Sign up' }}
        </button>

        <button type="button" class="google-btn" @click="handleGoogleAuth">
          <span class="google-icon">G</span>
          Continue with Google
        </button>

        <p class="toggle-form">
          {{ isLogin ? "Don't have an account?" : 'Already have an account?' }}
          <a href="#" @click.prevent="toggleForm">
            {{ isLogin ? 'Sign up' : 'Log in' }}
          </a>
        </p>
      </form>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use 'sass:color';
// Variables
$primary-color: #4285f4;
$error-color: #dc3545;
$border-color: #ddd;
$text-color: #333;
$text-secondary: #666;
$background-color: #f5f5f5;
$white: #fff;
$box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
$border-radius: 6px;
$transition-duration: 0.3s;

// Mixins
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin form-input {
  padding: 0.75rem;
  border: 1px solid $border-color;
  border-radius: $border-radius;
  font-size: 1rem;
  transition: border-color $transition-duration ease;
}

.auth-container {
  min-height: 100vh;
  @include flex-center;
  background-color: $background-color;
  padding: 1rem;
}

.auth-card {
  background: $white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: $box-shadow;
  width: 100%;
  max-width: 400px;
}

.auth-title {
  font-size: 2rem;
  text-align: center;
  margin-bottom: 2rem;
  color: $text-color;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;

  label {
    font-size: 0.9rem;
    color: $text-secondary;
  }

  input {
    @include form-input;

    &:focus {
      outline: none;
      border-color: $primary-color;
    }

    &.error {
      border-color: $error-color;
    }
  }
}

.password-input {
  position: relative;

  input {
    width: 100%;
    padding-right: 40px;
  }

  .toggle-password {
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    font-size: 1.2rem;
  }
}

.error-message {
  color: $error-color;
  font-size: 0.8rem;
}

%button-base {
  @include form-input;
  cursor: pointer;
  transition: all $transition-duration ease;
}

.submit-btn {
  @extend %button-base;
  background-color: $primary-color;
  color: $white;
  border: none;

  &:hover {
    background-color: color.adjust($primary-color, $lightness: -10%);
  }
}

.google-btn {
  @extend %button-base;
  @include flex-center;
  gap: 0.5rem;
  background-color: $white;
  color: $text-color;

  .google-icon {
    font-weight: bold;
    color: $primary-color;
  }

  &:hover {
    background-color: color.adjust($white, $lightness: -2%);
  }
}

.toggle-form {
  text-align: center;
  margin: 0;
  font-size: 0.9rem;
  color: $text-secondary;

  a {
    color: $primary-color;
    text-decoration: none;
    font-weight: 500;

    &:hover {
      text-decoration: underline;
    }
  }
}
</style>
