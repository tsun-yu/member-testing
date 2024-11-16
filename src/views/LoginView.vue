<template>
  <div class="login-container">
    <h1>Login</h1>
    <form @submit.prevent="singup">
      <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" id="email" v-model="email" required />
      </div>
      <div class="form-group">
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <button type="submit">sign up</button>
    </form>
    <form @submit.prevent="login">
      <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" id="email" v-model="email" required />
      </div>
      <div class="form-group">
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <button type="submit">Log in</button>
    </form>
    <button @click="checkStatus">check status</button>
    <button @click="signInWithGoogle">Google</button>
  </div>
</template>

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

const email = ref('')
const password = ref('')
const singup = async () => {
  try {
    const userCredential = await createUserWithEmailAndPassword(
      auth,
      email.value,
      password.value,
    )
    console.log('sign up', userCredential)
    sendEmailVerification(auth.currentUser)
  } catch (error) {
    console.error(error)
  }
}
const login = async () => {
  try {
    const userCredential = await signInWithEmailAndPassword(
      auth,
      email.value,
      password.value,
    )
    console.log('log in', userCredential)
  } catch (error) {
    console.error(error)
  }
}
const checkStatus = async () => {
  onAuthStateChanged(auth, user => {
    console.log(user)
  })
}
const signInWithGoogle = async () => {
  signInWithPopup(auth, provider)
    .then(result => {
      console.log('result', result)
      // This gives you a Google Access Token. You can use it to access the Google API.
      const credential = GoogleAuthProvider.credentialFromResult(result)
      console.log('credential', credential)
      const token = credential.accessToken
      console.log('token', token)
      // The signed-in user info.
      const user = result.user
      console.log('user', user)
      // IdP data available using getAdditionalUserInfo(result)
      // ...
    })
    .catch(error => {
      // Handle Errors here.
      const errorCode = error.code
      const errorMessage = error.message
      // The email of the user's account used.
      const email = error.customData.email
      // The AuthCredential type that was used.
      const credential = GoogleAuthProvider.credentialFromError(error)
      // ...
    })
}
</script>

<style scoped>
.login-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type='email'],
input[type='password'] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #0056b3;
}
</style>
