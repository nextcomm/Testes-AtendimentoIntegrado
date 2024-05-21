<template>
  <div
    class="h-full w-16 bg-white dark:bg-slate-900 border-r border-slate-50 dark:border-slate-800/50 rtl:border-l rtl:border-r-0 flex justify-between flex-col"
  >
    <div class="flex flex-col items-center">
      <logo
        :source="logoSource"
        :name="installationName"
        :account-id="accountId"
        class="m-4 mb-10"
      />
      <primary-nav-item
        v-for="menuItem in menuItems"
        :key="menuItem.toState"
        :icon="menuItem.icon"
        :name="menuItem.label"
        :to="menuItem.toState"
        :is-child-menu-active="menuItem.key === activeMenuItem"
      />
    </div>
    <div class="relative flex flex-col items-center justify-end pb-6">
      <SwitchUserCard
        :is-active="showBlankCard"
        @toggle-card="toggleBlankCard"
      />
      <!--
      <TypebotModal
        :is-active="showBlankCard"
        @toggle-card="toggleBlankCard"
      />
      -->
      <!--
      <Kuma
        :is-active="showBlankCard"
        @toggle-card="toggleBlankCard"
      />
      -->

      <!--
      <div @click="toggleToolsDropdown" class="cursor-pointer p-4">
        <img src="./toolss.png" alt="Tools Icon" class="w-6 h-6">
      </div>
      <div v-if="showToolsDropdown" class="dropdown-content">
        <a href="https://monitoramento-api.nextcomm.tech/dashboard" target="_blank" rel="noopener noreferrer" class="block p-2">Monitoramento Instâncias</a>
        <a href="http://example.com/option2" target="_blank" rel="noopener noreferrer" class="block p-2">Opção 2</a>
        <a href="http://example.com/option3" target="_blank" rel="noopener noreferrer" class="block p-2">Opção 3</a>
      </div>
      -->
      <notification-bell @open-notification-panel="openNotificationPanel" />
      <agent-details @toggle-menu="toggleOptions" />
      <options-menu
        :show="showOptionsMenu"
        @toggle-accounts="toggleAccountModal"
        @show-support-chat-window="toggleSupportChatWindow"
        @key-shortcut-modal="$emit('key-shortcut-modal')"
        @close="toggleOptions"
      />
    </div>
  </div>
</template>
<script>
import Logo from './Logo.vue';
import PrimaryNavItem from './PrimaryNavItem.vue';
import OptionsMenu from './OptionsMenu.vue';
import SwitchUserCard from './SwitchUserCard.vue';
// import TypebotModal from './TybebotModal.vue';
// import Kuma from './Kuma.vue';
import AgentDetails from './AgentDetails.vue';
import NotificationBell from './NotificationBell.vue';
import { frontendURL } from 'dashboard/helper/URLHelper';
import { ACCOUNT_EVENTS } from '../../../helper/AnalyticsHelper/events';
import { mapGetters } from 'vuex';

export default {
  components: {
    Logo,
    PrimaryNavItem,
    OptionsMenu,
    AgentDetails,
    NotificationBell,
    SwitchUserCard,
  },
  props: {
    teamState: {
      type: String,
      default: '',
    },
    isACustomBrandedInstance: {
      type: Boolean,
      default: false,
    },
    logoSource: {
      type: String,
      default: '',
    },
    installationName: {
      type: String,
      default: '',
    },
    accountId: {
      type: Number,
      default: 0,
    },
    menuItems: {
      type: Array,
      default: () => [],
    },
    activeMenuItem: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      showOptionsMenu: false,
      showBlankCard: false,
      showToolsDropdown: false,
    };
  },
  computed: {
    ...mapGetters({
      currentUser: 'getCurrentUser',
      currentAccountId: 'getCurrentAccountId',
    }),
  },
  watch: {
    $route() {
      // Executar a lógica necessária aqui
      this.listTeams();
    },
  },
  created() {
    this.listTeams();
  },
  methods: {
    frontendURL,
    toggleOptions() {
      this.showOptionsMenu = !this.showOptionsMenu;
    },
    toggleToolsDropdown() {
      this.showToolsDropdown = !this.showToolsDropdown; // Alterna a visibilidade do dropdown de TOOLS
    },
    async listTeams() {
      if (this.currentUser.role === 'administrator') {
        // ESTA OK NAO FAZ NADA SE FOR ADMIN
      } else {
        try {
          const response = await fetch(
            `http://5.78.98.87:3000/api/v1/accounts/${this.currentAccountId}/teams`,
            {
              headers: {
                api_access_token: this.currentUser.access_token,
              },
            }
          );
          const data = await response.json();
          // Percorrer o array de times para encontrar o primeiro time em que is_member é verdadeiro
          const firstMemberTeam = data.find(team => team.is_member === true);
          if (firstMemberTeam) {
            const firstMemberTeamId = firstMemberTeam.id;
            // Atualizar o toState do menuItem com a URL do time encontrado
            const conversationMenuItem = this.menuItems.find(
              item => item.key === 'conversations'
            );
            if (conversationMenuItem) {
              const newToState = `/app/accounts/${this.currentAccountId}/team/${firstMemberTeamId}`;
              // Utilizar this.$set para garantir a reatividade da propriedade
              this.$set(conversationMenuItem, 'toState', newToState);

              this.$forceUpdate();
            } else {
              // c o error('menuItem com a key "conversation" não encontrado.');
            }
          } else {
            // c o error('menuItem com a key "conversation" não encontrado.');
          }
        } catch (error) {
          // c o error('menuItem com a key "conversation" não encontrado.');
        }
      }
    },
    toggleAccountModal() {
      this.$emit('toggle-accounts');
    },
    toggleSupportChatWindow() {
      window.$chatwoot.toggle();
    },
    openNotificationPanel() {
      this.$track(ACCOUNT_EVENTS.OPENED_NOTIFICATIONS);
      this.$emit('open-notification-panel');
    },
    toggleBlankCard() {
      this.showBlankCard = !this.showBlankCard;
      window.location.reload();
    },
  },
};
</script>
<style>
.dropdown-content {
  position: absolute;
  left: 100%; /* Posiciona o dropdown à direita da sidebar */
  top: 0; /* Alinha o topo do dropdown com o botão Tools */
  min-width: 200px; /* Largura mínima para o dropdown */
  max-width: 300px; /* Largura máxima para evitar um dropdown excessivamente longo */
  background-color: #fff; /* Cor de fundo branca para o dropdown */
  box-shadow:
    0 4px 8px rgba(0, 0, 0, 0.25),
    0 6px 20px rgba(0, 0, 0, 0.19); /* Sombra para destacar o dropdown */
  border-radius: 5px; /* Bordas arredondadas */
  overflow: hidden; /* Esconde qualquer conteúdo que exceda o tamanho do dropdown */
  z-index: 1000; /* Eleva o dropdown acima de outros elementos */
}

.dropdown-content a {
  padding: 15px 20px; /* Espaçamento interno maior para os links */
  text-decoration: none; /* Remove sublinhado */
  display: block; /* Exibe os links em blocos */
  color: #333; /* Cor do texto escuro para melhor leitura */
  border-bottom: 1px solid #eee; /* Linha divisória entre os itens */
  transition: background-color 0.3s; /* Transição suave para o hover */
}

.dropdown-content a:last-child {
  border-bottom: none; /* Remove a borda do último item */
}

.dropdown-content a:hover {
  background-color: #f9f9f9; /* Cor de fundo ao passar o mouse */
}
</style>
