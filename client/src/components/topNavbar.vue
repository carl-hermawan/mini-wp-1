<template>
  <div id='container'>
    <b-navbar toggleable="md" type="dark" variant="info" >
      <b-navbar-brand>MiniWP</b-navbar-brand>

      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav v-show='isLogin' class='ml-auto'>
          <b-nav-item @click.prevent='newPost'>New Post</b-nav-item>
          <b-nav-item @click.prevent='myposts'>My Posts</b-nav-item>
          <b-nav-item @click.prevent='logout'>Logout</b-nav-item>
        </b-navbar-nav>

        <!-- Right aligned nav items -->
        <b-navbar-nav v-show='!isLogin' class="ml-auto">
          <b-nav-form>
            <b-form-input size="sm" class="mr-sm-2" placeholder="Email" v-model='email' type='email'></b-form-input>
            <b-form-input size="sm" class="mr-sm-2" placeholder="Password" v-model='password' type='password'></b-form-input>
            <b-button size="sm" class="my-2 mx-1 my-sm-0" type="submit" @click.prevent='login'>Login</b-button>
            <span>or</span>
            <b-button size="sm" class="my-2 mx-1 my-sm-0" type="submit" @click.prevent='register'>Register</b-button>
          </b-nav-form>
          <b-nav-item-dropdown text="Use other account" right>
            <div id="google-signin-button"></div>
          </b-nav-item-dropdown>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
  </div>
</template>

<script>
  import axios from '../config/axios.config.js'
  import Swal from 'sweetalert2'
export default {

  name: 'topNavbar',

  data () {
    return {
      isLogin: false,
      email: '',
      password: '',
    }
  },
  methods: {
    register(){
      axios({
        method: 'post',
        url: '/user/register',
        data: {
          email: this.email,
          password: this.password
        }
      })
        .then(({data}) => {
          Swal.fire('Success', 'Register success', 'success')
          this.login(false)
          this.email = ''
          this.password = ''
        })
        .catch((err) => {
          Swal.fire('Error', err.response.data.message, 'error')
        })
    },
    login(isRegistered){
      axios({
        method: 'post',
        url: '/user/login',
        data: {
          email: this.email,
          password: this.password
        }
      })
        .then(({data}) => {
          localStorage.setItem('access_token', data)
          if (isRegistered){
            Swal.fire('Success', 'Login success', 'success')
          }
          this.$emit('show', 'list')
          this.isLogin = true
          this.email = ''
          this.password = ''
        })
        .catch((err) => {
          Swal.fire('Error', err.response.data.message, 'error')
        })
    },
    onSignIn (googleUser) {
      const id_token = googleUser.getAuthResponse().id_token;
      axios({
        method: 'post',
        url: '/user/googleLogin',
        data: {
          token: id_token
        }
      })
        .then(({data}) => {
          localStorage.setItem('access_token', data)
          Swal.fire('Success', 'Login success', 'success')
          this.$emit('show', 'list')
          this.isLogin = true
          this.email = ''
          this.password = ''
        })
        .catch((err) => {
          Swal.fire('Error', err.response.data.message, 'error')
        })
    },
    logout(){
      this.isLogin = false
      let auth2 = gapi.auth2.getAuthInstance();
      auth2.signOut()
      localStorage.removeItem('access_token')
      this.email = ''
      this.password = ''
      this.$emit('show', '')
    },
    newPost(){
      this.$emit('show', 'form')
    },
    myposts(){
      this.$emit('show', 'list')
    }
  },
  created(){
    if(localStorage.getItem('access_token')){
      this.$emit('show', 'list')
      this.isLogin = true
    }
  },
  mounted(){
    gapi.signin2.render('google-signin-button', {
      onsuccess: this.onSignIn
    })
  }
}
</script>

<style scoped>
  .navbar-brand{
    cursor: default;
  }
  #google-signin-button{
    margin-left: 20px;
  }
  #container{
    position: sticky;
    top: 0;
    z-index: 1;
  }
</style>  