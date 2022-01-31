<template>
  <v-app>
    <v-main>
      <v-container class="mt-4">
        <v-row justify="center">
          <v-col cols="11" sm="6">
            <div id="google-sign-in"></div>
            <div v-if="googleProfile" class="mt-4">
              <h3>Google Profile:</h3>
              <ul>
                <li><b>ID:</b> {{ googleProfile.id }}</li>
                <li><b>Name:</b> {{ googleProfile.fullName }}</li>
              </ul>
              <v-btn class="mt-2" @click="googleSignOut">
                Google Sign Out
              </v-btn>
            </div>
          </v-col>
          <v-col cols="11" sm="6">
            <v-btn color="primary" @click="facebookSignIn">
              <v-icon dark left> mdi-facebook </v-icon>
              {{ facebookButtonText }}
            </v-btn>
            <div v-if="facebookProfile" class="mt-4">
              <h3>Facebook Profile:</h3>
              <ul>
                <li><b>ID:</b> {{ facebookProfile.id }}</li>
                <li><b>Name:</b> {{ facebookProfile.name }}</li>
              </ul>
              <v-btn class="mt-2" @click="facebookSignOut">
                Facebook Sign Out
              </v-btn>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    googleProfile: null,
    facebookProfile: null
  }),

  computed: {
    facebookButtonText() {
      return this.facebookProfile
        ? "Logged in with Facebook"
        : "Log in with Facebook";
    }
  },

  created() {
    this.initGoogleAuth();
    this.initFacebookAuth();
  },

  mounted() {
    this.renderGoogleAuthButton();
    this.checkFacebookLoginStatus();
  },

  methods: {
    initGoogleAuth() {
      window.gapi.load("auth2", function () {
        window.gapi.auth2.init({
          client_id: import.meta.env.VITE_GOOGLE_CLIENT_ID
        });
      });
    },

    renderGoogleAuthButton() {
      window.gapi.signin2.render("google-sign-in", {
        scope: "profile email",
        width: 250,
        height: 40,
        longtitle: true,
        theme: "dark",
        onsuccess: this.onGoogleSignIn
      });
    },

    onGoogleSignIn(user) {
      const profile = user.getBasicProfile();
      const id = profile.getId();
      const fullName = profile.getName();
      this.googleProfile = { id, fullName };
    },

    googleSignOut() {
      var auth2 = window.gapi.auth2.getAuthInstance();
      auth2.signOut().then(() => {
        this.googleProfile = null;
      });
    },

    initFacebookAuth() {
      window.FB.init({
        appId: import.meta.env.VITE_FACEBOOK_CLIENT_ID,
        cookie: true, // Enable cookies to allow the server to access the session.
        xfbml: true, // Parse social plugins on this webpage.
        version: "v12.0" // Use this Graph API version for this call.
      });
    },

    facebookSignIn() {
      if (!this.facebookProfile) {
        window.FB.login((response) => {
          if (response.authResponse) {
            window.FB.api("/me", (profile) => {
              this.facebookProfile = profile;
            });
          }
        });
      }
    },

    checkFacebookLoginStatus() {
      const self = this;
      window.FB.getLoginStatus(function (response) {
        if (response.status === "connected") {
          window.FB.api("/me", (profile) => {
            self.facebookProfile = profile;
          });
        } else {
          self.facebookSignIn();
        }
      });
    },

    facebookSignOut() {
      window.FB.logout(() => {
        this.facebookProfile = null;
      });
    }
  }
};
</script>
