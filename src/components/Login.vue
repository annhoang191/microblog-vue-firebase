<template>
  <div id="login">
    <transition name="fade">
      <div v-if="performingRequest" class="loading">
        <p>Loading...</p>
      </div>
    </transition>
    <section>
      <div class="col1">
        <h1>Vuegram</h1>
        <p>Welcome to the <a href="https://savvyapps.com/" target="_blank">Savvy Apps</a> sample social media web app powered by Vue.js and Firebase.
          Build this project by checking out The Definitive Guide to Getting Started with Vue.js
        </p>
      </div>
      <div class="col2" :class="{'signup-form': !showLoginForm && !showForgotPassword}">
        <form v-if="showLoginForm" @submit.prevent>
          <h1>Welcome Back</h1>
          <label for="email1">Email</label>
          <input v-model.trim="loginForm.email"
            type="text" placeholder="you@email.com" id="email1" />
          <label for="password1">Password</label>
          <input v-model.trim="loginForm.password"
            type="password" placeholder="******" id="password1" />
          <button @click="login" class="button">Log In</button>
          <div class="extras">
            <a @click="togglePasswordReset">Forgot Password</a>
            <a @click="toggleForm">Create an Account</a>
          </div>
        </form>

        <form v-if="!showLoginForm  && !showForgotPassword" @submit.prevent>
          <h1>Get started</h1>

          <label for="name">Name</label>
          <input v-model.trim="signupForm.name" type="text" placeholder="Savvy Apps" id="name" />

          <label for="title">Title</label>
          <input v-model.trim="signupForm.title" type="text" placeholder="Company" id="title" />

          <label for="email2">Email</label>
          <input v-model.trim="signupForm.email" type="text" placeholder="you@email.com" id="email2" />

          <label for="password2">Password</label>
          <input v-model.trim="signupForm.password" type="password" placeholder="min 6 characters" id="password2" />

          <button @click="signup" class="button">Sign Up</button>

          <div class="extras">
            <a @click="toggleForm">Back to Log In</a>
          </div>
        </form>

        <form v-if="showForgotPassword" @submit.prevent class="password-reset">
          <div v-if="!passwordResetSuccess">
            <h1>Reset Password</h1>
            <p>Will send you an email to reset password</p>

            <label for="email">Email</label>
            <input v-model.trim="passwordForm.email"
              type="text" placeholder="youremail@example.com" id="email3">

            <button @click="resetPassword" class="button">Submit</button>
            <div class="extras">
              <a @click="togglePasswordReset">Back to login</a>
            </div>
          </div>
          <div v-else >
            <h1>Email sent</h1>
            <p>Please check your email</p>
            <button @click="togglePasswordReset">Back to login</button>
          </div>
        </form>
        <transition name="fade">
          <div v-if="errorMsg !== ''" class="error-msg">
            <p>{{errorMsg}}</p>
          </div>
        </transition>
      </div>
    </section>
  </div>
</template>

<script>
const fb = require('../firebaseConfig.js')

export default {
  data() {
    return {
      loginForm: {
        email: '',
        password: ''
      },
      signupForm: {
        name: '',
        title: '',
        email: '',
        password: ''
      },
      passwordForm: {
        email: ''
      },
      showLoginForm: true,
      showForgotPassword: false,
      passwordResetSuccess: false,
      performingRequest: false,
      errorMsg: ''
    }
  },

  methods: {
    login() {
      this.performingRequest = true

      fb.auth.signInWithEmailAndPassword(this.loginForm.email, this.loginForm.password).then(user => {
        this.$store.commit('setCurrentUser', user.user)
        this.$store.dispatch('fetchUserProfile')
        this.performingRequest = false
        this.$router.push('/dashboard')
      }).catch(err => {
        console.log(err)
        this.performingRequest = false
        this.errorMsg = err.message
      })
    },

    signup() {
      this.performingRequest = true

      fb.auth.createUserWithEmailAndPassword(this.signupForm.email, this.signupForm.password).then(user => {
        this.$store.commit('setCurrentUser', user.user)

        fb.usersCollection.doc(user.user.uid).set({
          name: this.signupForm.name,
          title: this.signupForm.title
        }).then(() => {
          this.$store.dispatch('fetchUserProfile')
          this.performingRequest = false
          this.$router.push('/dashboard')
        }).catch(err => {
          console.log(err)
          this.performingRequest = false
          this.errorMsg = err.message
        })
      }).catch(err => {
        console.log(err)
      })
    },

    toggleForm() {
      this.errorMsg = ''
      this.showLoginForm = !this.showLoginForm
    },

    resetPassword() {
      this.performingRequest = true

      fb.auth.sendPasswordResetEmail(this.passwordForm.email).then(() => {
        this.performingRequest = false
        this.passwordResetSuccess = true
        this.passwordForm.email = ''
      }).catch(err => {
        this.performingRequest = false
        this.errorMsg = err.message
      })
    },

    togglePasswordReset() {
      if(this.showForgotPassword) {
        this.showForgotPassword = false
        this.showLoginForm = true
        this.passwordResetSuccess = false
      } else {
        this.showLoginForm = false
        this.showForgotPassword = true
      }
    }
  }
}
</script>
