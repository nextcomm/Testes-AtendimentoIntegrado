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
    <div class="flex flex-col items-center justify-end pb-6">
      <SwitchUserCard
        :is-active="showBlankCard"
        @toggle-card="toggleBlankCard"
      />
      <primary-nav-item
        v-if="!isACustomBrandedInstance"
        icon="book-open-globe"
        name="DOCS"
        :open-in-new-page="true"
        :to="helpDocsURL"
      />
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
import AgentDetails from './AgentDetails.vue';
import NotificationBell from './NotificationBell.vue';
import wootConstants from 'dashboard/constants/globals';
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
      helpDocsURL: wootConstants.DOCS_URL,
      showOptionsMenu: false,
      showBlankCard: false,
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
    async listTeams() {
      if (this.currentUser.role === 'administrator') {
        // ESTA OK NAO FAZ NADA SE FOR ADMIN
      } else {
        try {
          const response = await fetch(
            `http://5.78.105.86:3000/api/v1/accounts/${this.currentAccountId}/teams`,
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
  beforeRouteUpdate(to, from, next) {
    // Chamar listTeams toda vez que a rota é atualizada
    this.listTeams();
    next();
  },
};
</script>
