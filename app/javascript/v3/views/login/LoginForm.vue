<template>
    <div>
      <div
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 modal-mask"
        @click="closeLoginCardOutside"
      >
        <div class="modal-card" @click.stop>
          <div
            class="p-8 bg-white dark:bg-slate-800 shadow-md rounded-lg login-fform"
          >
            <!-- Botão Go Back -->
            <button class="text-gray-500 hover:text-gray-700" @click="goBack">
              <fluent-icon icon="chevron-left" class="h-6 w-6" />
            </button>
            <form class="space-y-5" @submit.prevent="submitLogin">
              <form-input
                v-model.trim="credentials.email"
                name="email_address"
                type="text"
                data-testid="email_input"
                :tabindex="1"
                required
                :label="$t('LOGIN.EMAIL.LABEL')"
                :placeholder="$t('LOGIN.EMAIL.PLACEHOLDER')"
                :has-error="$v.credentials.email.$error"
                @input="$v.credentials.email.$touch"
              />
              <form-input
                v-model.trim="credentials.password"
                type="password"
                name="password"
                data-testid="password_input"
                required
                :tabindex="2"
                :label="$t('LOGIN.PASSWORD.LABEL')"
                :placeholder="$t('LOGIN.PASSWORD.PLACEHOLDER')"
                :has-error="$v.credentials.password.$error"
                @input="$v.credentials.password.$touch"
              />
              <submit-button
                :disabled="loginApi.showLoading"
                :tabindex="3"
                :button-text="$t('LOGIN.SUBMIT')"
                :loading="loginApi.showLoading"
              />
            </form>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { required, email } from 'vuelidate/lib/validators';
  import { mapGetters } from 'vuex';
  import { parseBoolean } from '@chatwoot/utils';
  import FormInput from '../../components/Form/Input.vue';
  import { login } from '../../api/auth';
  import SubmitButton from '../../components/Button/SubmitButton.vue';
  
  const ERROR_MESSAGES = {
    'no-account-found': 'LOGIN.OAUTH.NO_ACCOUNT_FOUND',
    'business-account-only': 'LOGIN.OAUTH.BUSINESS_ACCOUNTS_ONLY',
  };
  
  export default {
    components: {
      FormInput,
      SubmitButton,
    },
    props: {
      ssoAuthToken: { type: String, default: '' },
      ssoAccountId: { type: String, default: '' },
      ssoConversationId: { type: String, default: '' },
      config: { type: String, default: '' },
      email: { type: String, default: '' },
      authError: { type: String, default: '' },
    },
    data() {
      return {
        credentials: {
          email: this.email, // Alterado para atribuir o email diretamente da propriedade
          password: '',
        },
        loginApi: {
          message: '',
          showLoading: false,
          hasErrored: false,
        },
        showLoginCard: true,
      };
    },
    validations: {
      credentials: {
        password: {
          required,
        },
        email: {
          required,
          email,
        },
      },
    },
    computed: {
      ...mapGetters({ globalConfig: 'globalConfig/get' }),
      showGoogleOAuth() {
        return Boolean(window.chatwootConfig.googleOAuthClientId);
      },
      showSignupLink() {
        return parseBoolean(window.chatwootConfig.signupEnabled);
      },
    },
    created() {
      if (this.ssoAuthToken) {
        this.submitLogin();
      }
      if (this.authError) {
        const message = ERROR_MESSAGES[this.authError] ?? 'LOGIN.API.UNAUTH';
        this.showAlert(this.$t(message));
        window.requestIdleCallback(() => {
          const { query } = this.$route;
          this.$router.replace({ query: { ...query, error: undefined } });
        });
      }
      if (this.$route.query.email) {
        this.credentials.email = decodeURIComponent(this.$route.query.email); // Atualizado para atribuir ao campo de email do formulário
      }
    },
    methods: {
      showAlert(message) {
        this.loginApi.showLoading = false;
        this.loginApi.message = message;
        bus.$emit('newToastMessage', this.loginApi.message);
      },
      submitLogin() {
        if (this.$v.credentials.email.$invalid && !this.email) {
          this.showAlert(this.$t('LOGIN.EMAIL.ERROR'));
          return;
        }
  
        this.loginApi.hasErrored = false;
        this.loginApi.showLoading = true;
  
        const credentials = {
          email: this.credentials.email,
          password: this.credentials.password,
          sso_auth_token: this.ssoAuthToken,
          ssoAccountId: this.ssoAccountId,
          ssoConversationId: this.ssoConversationId,
        };
  
        login(credentials)
          .then(() => {
            this.showAlert(this.$t('LOGIN.API.SUCCESS_MESSAGE'));
          })
          .catch(response => {
            if (this.email) {
              window.location = '/app/login';
            }
            this.loginApi.hasErrored = true;
            this.showAlert(response?.message || this.$t('LOGIN.API.UNAUTH'));
          });
      },
      closeLoginCard(event) {
        if (event.target === event.currentTarget) {
          this.showLoginCard = false;
        }
      },
      closeLoginCardOutside(event) {
        if (!event.target.closest('.modal-card')) {
          this.showLoginCard = false;
        }
      },
      goBack() {
        // Emitir um evento para informar ao pai que o usuário quer voltar
        this.$emit('back');
      },
    },
  };
  </script>
  
  <style scoped>
  .modal-card {
    max-width: 400px;
    margin: auto;
  }
  
  /* Ajuste de posição do botão Go Back */
  .modal-card button {
    top: 0;
    left: 0;
    margin-top: 10px; /* Ajuste conforme necessário */
    margin-left: 10px; /* Ajuste conforme necessário */
  }
  
  /* Adicione um cursor pointer ao botão para indicar que é clicável */
  .modal-card button:hover {
    cursor: pointer;
  }
  
  .login-fform {
    width: 500px;
  }
  </style>
  