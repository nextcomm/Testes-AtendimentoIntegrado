<template>
  <div>
    <button
      class="switch-btn text-slate-600 dark:text-slate-100 w-10 h-10 my-2 flex items-center justify-center rounded-lg hover:bg-slate-25 dark:hover:bg-slate-700 dark:hover:text-slate-100 hover:text-slate-600 relative"
      :class="{
        'bg-woot-50 dark:bg-slate-800 text-woot-500 hover:bg-woot-50': isActive,
      }"
      @click="toggleCard"
    >
      <fluent-icon
        icon="person-assign"
        :class="{ 'text-woot-500': isActive }"
      />
    </button>
    <div
      v-if="showModal"
      class="fixed inset-0 flex items-center justify-center modal-mask"
      @click="closeModalOutside"
    >
      <div class="modal-overlay" />
      <div class="modal-card-wrapper">
        <div
          v-if="!showLoginForm"
          class="modal-card bg-white dark:bg-slate-800 shadow-md rounded-lg overflow-hidden"
          @click.stop
        >
          <div class="grid grid-cols-3 gap-4 p-4 max-h-96 overflow-y-auto">
            <div
              v-for="(user, index) in userData"
              :key="index"
              class="user-card"
              @click="handleUserNameClick(user.email)"
            >
              <img :src="getUserAvatar(user)" class="user-avatar" />
              <p class="text-center user-name">{{ user.name }}</p>
            </div>
          </div>
          <button
            class="absolute top-0 right-0 p-2 text-gray-500 hover:text-gray-700"
            @click="closeModal"
          >
            <fluent-icon icon="close" class="h-6 w-6" />
          </button>
        </div>
      </div>
    </div>
    <LoginForm
      v-if="showLoginForm"
      :email="selectedUserEmail"
      @back="(showLoginForm = false), (showModal = true)"
    />
  </div>
</template>

<script>
import LoginForm from '../../../../v3/views/login/LoginForm.vue';

export default {
  components: {
    LoginForm,
  },
  props: {
    isActive: {
      type: Boolean,
      default: false,
    },
    currentUser: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      showModal: false,
      showLoginForm: false,
      openedByTimer: false, // Adicione esta variável de controle
      userData: [],
      logoutTimer: null, // Adicione o timer de logout
      selectedUserEmail: '', // Adicione o e-mail do usuário selecionado
    };
  },
  created() {
    this.startLogoutTimer();
  },
  beforeDestroy() {
    clearInterval(this.logoutTimer);
  },
  methods: {
    toggleCard() {
      this.showModal = !this.showModal;
      if (this.showModal) {
        this.fetchUserNames();
      }
    },
    closeModalOutside(event) {
      if (!this.openedByTimer && !event.target.closest('.modal-card')) {
        this.closeModal();
      }
    },
    handleUserNameClick(email) {
      // Defina o e-mail do usuário selecionado
      this.selectedUserEmail = email;
      // Emita o evento para mostrar o formulário de login
      this.$emit('show-login-form');
      this.$emit('user-clicked', email);
      // Alterne a visibilidade do formulário de login
      this.showLoginForm = true;
      this.showModal = false; // Fechar a modal após clicar no usuário
    },
    async fetchUserNames() {
      try {
        const response = await fetch(
          'http://5.78.105.86:3000/api/v1/accounts/1/agents',
          {
            headers: {
              api_access_token: 'mNziGJGSrxGEV3oUtF8F6oYJ',
            },
          }
        );
        const data = await response.json();
        this.userData = data;
      } catch (error) {
        // onsole.error('Erro ao buscar nomes de usuários:', error);
      }
    },
    startLogoutTimer() {
      this.fetchUserNames();
      // Inicia o cronômetro para chamar Auth.logout() a cada 5 minutos
      this.logoutTimer = setInterval(
        () => {
          this.showModal = true; // Abre a modal automaticamente após 10 segundos
          this.openedByTimer = true; // Defina openedByTimer como true quando aberto pelo time
        },
        100 * 60 * 1000
      ); // 10 segundos em milissegundos
    },
    enterFullscreen() {
      const element = document.documentElement; // Seleciona o elemento raiz (a página inteira)
      if (element.requestFullscreen) {
        element.requestFullscreen(); // Entra em modo de tela cheia
      } else if (element.mozRequestFullScreen) {
        element.mozRequestFullScreen(); // Firefox
      } else if (element.webkitRequestFullscreen) {
        element.webkitRequestFullscreen(); // Chrome, Safari e Opera
      } else if (element.msRequestFullscreen) {
        element.msRequestFullscreen(); // Internet Explorer
      }
    },
    getUserAvatar(user) {
      // Verifica se o usuário tem um avatar
      if (user.thumbnail && user.thumbnail !== '') {
        return user.thumbnail;
      }
      // Caso contrário, retorna o caminho da imagem padrão
      return '/assets/administrate/user/avatar.png';
    },
    closeModal() {
      this.showModal = false;
    },
  },
};
</script>

<style scoped>
.modal-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.modal-card-wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.modal-card {
  width: 700px;
  height: 400px;
  box-shadow: 9px 7px 5px rgba(50, 50, 50, 0.77);
}

.user-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px; /* Espaçamento entre os cards */
}

.user-avatar {
  width: 100px;
  cursor: pointer;
  height: 100px;
  border-radius: 50%; /* Faz com que a imagem do avatar seja redonda */
  object-fit: cover; /* Redimensiona a imagem para caber no círculo */
}

.user-name {
  font-size: 16px;
  cursor: pointer;
}

.user-name:hover {
  color: #3182ce;
}

.switch-btn {
  cursor: pointer;
}

.user-card:hover {
  transform: scale(1.05); /* Aumenta o tamanho em 5% */
  transition: transform 0.3s ease; /* Adiciona uma transição suave */
}
</style>
