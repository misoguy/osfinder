<template>
  <v-container grid-list-md id="login">
    <v-flex xs12 sm8 offset-sm2 md6 offset-md3 lg4 offset-lg4>
      <v-card>
        <v-card-title>
          <v-layout column>
            <v-flex>
              <h3>Login via GitHub</h3>
              <v-btn
                block
                color="secondary"
                @click="githubLogin"
              >
                <img src="../assets/GitHub.png" alt="Github Login" />
                GITHUB
              </v-btn>
            </v-flex>
          </v-layout>
        </v-card-title>
        <v-divider />
        <v-card-title>
          <v-layout column>
            <v-flex>
              <h3>Login with Personal access tokens</h3>
              <v-text-field
                label="Personal access tokens"
                placeholder="Enter Personal access tokens"
                v-model="token"
              >
              </v-text-field>
              <v-btn @click="login" block outline color="primary">
                LOGIN
              </v-btn>
              <v-btn
                block
                outline
                href="https://github.com/settings/tokens/new"
                target="_blank"
                rel="noopener"
              >
                <img src="../assets/GitHub-black.png" alt="Get Token" />
                Get token from Github
              </v-btn>
              <v-layout row wrap justify-center align-center>
                <h4 class="body-2">REQUIRED SCOPES:</h4>
                <div>
                  <v-chip disabled outline label color="blue-grey">
                    <v-icon>public</v-icon>
                    <h4>public_repo</h4>
                  </v-chip>
                  <v-chip disabled outline label color="blue-grey">
                    <v-icon>face</v-icon>
                    <h4>read:user</h4>
                  </v-chip>
                </div>
              </v-layout>
            </v-flex>
          </v-layout>
        </v-card-title>
      </v-card>
    </v-flex>
  </v-container>
</template>

<script lang="ts">
import Vue from 'vue';
import netlify from 'netlify-auth-providers';

export default Vue.extend({
  name: 'Login',
  data() {
    return {
      token: '',
    };
  },
  methods: {
    login() {
      this.$store.dispatch('login', this.token).then(() => {
        this.$router.push('repositories');
      });
    },
    githubLogin() {
      const authenticator = new netlify({});
      authenticator.authenticate(
        { provider: 'github', scope: 'read:user, public_repo' },
        (err: any, data: any) => {
          if (err) {
            return alert('LOGIN ERROR');
          }
          const { token } = data;
          this.$store.dispatch('login', token).then(() => {
            this.$router.push('repositories');
          });
        }
      );
    },
  },
});
</script>

<style scoped>
h6 {
  text-align: center;
}

img {
  width: 1.4rem;
  margin-right: 0.5rem;
}
</style>
