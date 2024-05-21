<template>
  <div class="relative">
    <div
      class="flex px-4 pb-1 flex-row gap-1 pt-2.5 border-b border-transparent"
    >
      <woot-sidemenu-icon
        size="tiny"
        class="relative top-0 ltr:-ml-1.5 rtl:-mr-1.5"
      />
      <router-link
        :to="searchUrl"
        class="search-link flex-1 items-center gap-1 text-left h-6 rtl:mr-3 rtl:text-right rounded-md px-2 py-0 bg-slate-25 dark:bg-slate-800 inline-flex"
      >
        <div class="flex">
          <fluent-icon
            icon="search"
            class="search--icon text-slate-800 dark:text-slate-200"
            size="16"
          />
        </div>
        <p
          class="search--label mb-0 overflow-hidden whitespace-nowrap text-ellipsis text-sm text-slate-800 dark:text-slate-200"
        >
          {{ $t('CONVERSATION.SEARCH_MESSAGES') }}
        </p>
      </router-link>
      <switch-layout
        :is-on-expanded-layout="isOnExpandedLayout"
        @toggle="$emit('toggle-conversation-layout')"
      />
    </div>
  </div>
</template>

<script>
import { mixin as clickaway } from 'vue-clickaway';
import { mapGetters } from 'vuex';
import timeMixin from '../../../../mixins/time';
import messageFormatterMixin from 'shared/mixins/messageFormatterMixin';
import SwitchLayout from './SwitchLayout.vue';
import { frontendURL } from 'dashboard/helper/URLHelper';
export default {
  components: {
    SwitchLayout,
  },
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      },
    },
  },
  mixins: [timeMixin, messageFormatterMixin, clickaway],
  props: {
    isOnExpandedLayout: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    ...mapGetters({
      accountId: 'getCurrentAccountId',
    }),
    searchUrl() {
      return frontendURL(`accounts/${this.accountId}/search`);
    },
  },
};
</script>
<style lang="scss" scoped>
.search-link {
  background-color: #f1f5f9; // Cor de fundo mais clara para maior contraste
  border: 2px solid #cbd5e1; // Adicionar uma borda para definição

  &:hover {
    .search--icon,
    .search--label {
      @apply text-woot-700 dark:text-woot-700;
    }
  }

  .search--icon {
    color: #1f2937; // Cor mais escura para o ícone
  }

  .search--label {
    font-size: 1rem; // Aumentar o tamanho do texto
    color: #1f2937; // Cor mais escura para o texto
  }

  &.dark {
    background-color: #1e293b; // Cor de fundo mais escura para o tema escuro
    border: 2px solid #374151; // Adicionar uma borda para definição no tema escuro

    .search--icon {
      color: #f8fafc; // Cor mais clara para o ícone no tema escuro
    }

    .search--label {
      color: #f8fafc; // Cor mais clara para o texto no tema escuro
    }
  }
}
</style>
