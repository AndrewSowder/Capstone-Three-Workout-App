<template>
<div class="main">
  <div id="login" class="testbox">
    <form v-if= "loggingIn === false" class="form-signin" @submit.prevent="login">
      <h1 class="h3 mb-3 font-weight-normal">Please Sign In</h1>
      <div class="alert" role="alert" v-if="invalidCredentials">
        Invalid username and password!
      </div>
      <div
        class="alert-success"
        role="alert"
        v-if="this.$route.query.registration"
      >
        Thank you for registering, please sign in.
      </div>

      <label id="icon" for="name">
        <i><font-awesome-icon icon="fa-user" /></i>
      </label>
      <input
        type="text"
        name="name"
        id="name"
        autocomplete="off"
        placeholder="Username"
        v-model="user.username"
        required
        autofocus
      />

      <label id="icon" for="password">
        <i> <font-awesome-icon icon="fa fa-key" /></i>
      </label>
      <input
        type="password"
        name="password"
        id="password"
        placeholder="Password"
        v-model="user.password"
        required
      />
      <div class="send">
        <button type="submit">Sign In</button>
        <router-link :to="{ name: 'register' }">Need an account?</router-link>
      </div>
    </form>
    <img id="loading" v-else src="/WorkoutImages/output-onlinegiftools.gif" />
  </div>
  </div>
</template>

<script>
import authService from "../services/AuthService";

export default {
  name: "login",
  components: {},
  data() {
    return {
      loggingIn: false,
      user: {
        username: "",
        password: "",
      },
      invalidCredentials: false,
    };
  },
  methods: {
    login() {
      this.loggingIn = true;
      authService
        .login(this.user)
        .then((response) => {
          this.loggingIn = false;
          if (response.status == 200) {
            this.$store.commit("SET_AUTH_TOKEN", response.data.token);
            this.$store.commit("SET_USER", response.data.user);
            console.log(response.data.user.id)
            const id = response.data.user.id
            if (this.isTrainer(response.data.user.authorities[0].name)) {
              this.$router.push({ name: "trainer", params:{id: id} });
            } else if (this.isUser(response.data.user.authorities[0].name)) {
              this.$router.push({ name: "user", params:{id: id} });
            } else {
              this.$router.push({ name: "login" });
            }
          }
        })
        .catch((error) => {
          const response = error.response;
          this.loggingIn = false;

          if (response.status === 401) {
            this.invalidCredentials = true;
          }
        });
    },
    isTrainer(authorities) {
      if (authorities === "ROLE_TRAINER") {
        return true;
      }
    },
    isUser(authorities) {
      if (authorities === "ROLE_USER") {
        return true;
      }
    },
  },
};
</script>
<style scoped>
h1 {
  font-size: 32px;
  font-weight: 300;
  color: #4c4c4c;
  text-align: center;
  padding-top: 10px;
  margin-bottom: 10px;
}

.main{
  min-width: 310px;
  display: flex;
  justify-content: center;
  max-height:max-content;

}

.testbox {
  margin: 10px auto;
  max-width: 310px;
  min-width: 300px;
  height: 465px;
  border-radius: 8px/7px;
  background-color: #ebebeb;
  box-shadow: 1px 2px 5px black;
  border: solid 1px #cbc9c9;
}

#loading {
  width: 250px;
  height: 250px;
  margin-left: 35px;
  margin-top: 100px;
}

.alert {
  box-shadow: 1px 2px 5px black;
  text-align: center;
  font-size: 14px;
  font-weight: 600;
  background-color: rgb(240, 52, 52);
  color: #ebebeb;
  border-radius: 5px;
  animation: shake;
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, 0, 0);
  backface-visibility: hidden;
  perspective: 1000px;
}
.alert-success {
  box-shadow: 1px 2px 5px black;
  text-align: center;
  font-size: 14px;
  font-weight: 600;
  background-color: #3a57af;
  color: #ebebeb;
  border-radius: 5px;
 
  backface-visibility: hidden;
  perspective: 1000px;
}

@keyframes shake {
  10%,
  90% {
    transform: translate3d(-1px, 0, 0);
  }

  20%,
  80% {
    transform: translate3d(2px, 0, 0);
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, 0, 0);
  }

  40%,
  60% {
    transform: translate3d(4px, 0, 0);
  }
}

hr {
  color: #f86c6c;
  opacity: 0.3;
}

form {
  margin: 0 30px;
}

input[type="text"],
input[type="password"] {
  width: 180px;
  height: 39px;
  border-radius: 0px 4px 4px 0px/5px 5px 4px 4px;
  background-color: #fff;
  box-shadow: 1px 2px 5px black;
  border: solid 1px #cbc9c9;
  margin-left: -5px;
  margin-top: 13px;
  padding-left: 10px;
}

input {
  margin-bottom: 15px;
}

#icon {
  display: inline-block;
  width: 40px;
  background-color: #3a57af;
  padding: 8px 8px 8px 12px;
  margin-left: 15px;
  margin-right: 10px;
  color: white;
  box-shadow: 1px 2px 5px rgba(0, 0, 0, 0.09);
  border: solid 0px #cbc9c9;
}


.send {
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}

button {
  font-size: 14px;
  font-weight: 600;
  color: white;
  text-decoration: none;
  width: 100px;
  height: 30px;
  border-radius: 5px;
  background-color: #3a57af;
  box-shadow: 0 3px rgba(58, 87, 175, 0.75);
  align-self: center;
  margin-bottom: 10px;
}

button :hover {
  top: 3px;
  background-color: black;
  border: #4c4c4c;
}
</style>
