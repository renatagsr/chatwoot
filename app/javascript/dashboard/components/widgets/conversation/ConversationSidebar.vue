<script setup>
import { computed, ref } from 'vue';
import CopilotContainer from '../../copilot/CopilotContainer.vue';
import ContactPanel from 'dashboard/routes/dashboard/conversation/ContactPanel.vue';
import TabBar from 'dashboard/components-next/tabbar/TabBar.vue';
import { useI18n } from 'vue-i18n';
import { useMapGetter } from 'dashboard/composables/store';
import { FEATURE_FLAGS } from '../../../featureFlags';

// NEW: importe seu componente
import PipedriveTab from './PipedriveTab.vue';

const props = defineProps({
  currentChat: {
    required: true,
    type: Object,
  },
});

const emit = defineEmits(['toggleContactPanel']);

const { t } = useI18n();

const channelType = computed(() => props.currentChat?.meta?.channel || '');

// NEW: adicione "PIPEDRIVE" na lista de abas
// (cada objeto gerará uma aba no TabBar)
const CONTACT_TABS_OPTIONS = [
  { key: 'CONTACT', value: 'contact' },
  { key: 'COPILOT', value: 'copilot' },
  { key: 'PIPEDRIVE', value: 'pipedrive' },
];

const tabs = computed(() => {
  return CONTACT_TABS_OPTIONS.map(tab => ({
    // se quiser usar i18n, crie a key no seu arquivo de tradução.
    // Ou use label fixo, ex.: label: 'Pipedrive'.
    label: t(`CONVERSATION.SIDEBAR.${tab.key}`),
    value: tab.value,
  }));
});
const activeTab = ref(0);
const toggleContactPanel = () => {
  emit('toggleContactPanel');
};

const handleTabChange = selectedTab => {
  activeTab.value = tabs.value.findIndex(
    tabItem => tabItem.value === selectedTab.value
  );
};

// Feature flag do Copilot, se não precisar disto, você pode remover
const currentAccountId = useMapGetter('getCurrentAccountId');
const isFeatureEnabledonAccount = useMapGetter(
  'accounts/isFeatureEnabledonAccount'
);
const showCopilotTab = computed(() =>
  isFeatureEnabledonAccount.value(currentAccountId.value, FEATURE_FLAGS.CAPTAIN)
);
</script>

<template>
  <div
    class="ltr:border-l rtl:border-r border-n-weak h-full overflow-hidden z-10 min-w-[320px] w-[320px] 2xl:min-w-96 2xl:w-96 flex flex-col bg-n-background"
  >
    <!-- Se quiser exibir SEMPRE o TabBar, basta remover o v-if do showCopilotTab -->
    <div class="p-2">
      <TabBar
        :tabs="tabs"
        :initial-active-tab="activeTab"
        class="w-full [&>button]:w-full"
        @tab-changed="handleTabChange"
      />
    </div>

    <div class="flex flex-1 overflow-auto">
      <!-- Exibe o painel de contato quando a aba for 'contact' (índice 0) -->
      <ContactPanel
        v-if="activeTab === 0"
        :conversation-id="currentChat.id"
        :inbox-id="currentChat.inbox_id"
        :on-toggle="toggleContactPanel"
      />

      <!-- Exibe o Copilot somente se a aba for 'copilot' (índice 1) e o recurso estiver habilitado -->
      <CopilotContainer
        v-else-if="activeTab === 1 && showCopilotTab"
        :key="currentChat.id"
        :conversation-inbox-type="channelType"
        :conversation-id="currentChat.id"
        class="flex-1"
      />

      <!-- NEW: Exibe o PipedriveTab na aba 'pipedrive' (índice 2) -->
      <PipedriveTab
        v-else-if="activeTab === 2"
        :conversation="currentChat"
        class="flex-1"
      />
    </div>
  </div>
</template>
