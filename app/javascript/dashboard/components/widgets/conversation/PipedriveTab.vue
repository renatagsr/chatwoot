<template>
  <div class="p-4">
    <h3 class="text-lg font-bold mb-2">Pipedrive</h3>
    <div v-if="error" class="text-red-600">
      Erro: {{ error }}
    </div>
    <div v-else-if="dealInfo">
      <p><strong>Nome:</strong> {{ dealInfo.nome }}</p>
      <p><strong>Estágio:</strong> {{ dealInfo.estagio }}</p>
      <!-- Exemplo de botão -->
      <button
        class="bg-blue-500 text-white px-3 py-1 mt-2 rounded"
        @click="mudarEstagio"
      >
        Mudar estágio
      </button>
    </div>
    <div v-else>Carregando...</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  conversation: {
    type: Object,
    required: true,
  },
});

const dealInfo = ref(null);
const error = ref(null);

onMounted(async () => {
  try {
    // Exemplo: busca o telefone do cliente
    const phone = props.conversation.meta?.sender?.phone_number || '';
    // Faz chamada ao seu endpoint do n8n, por ex.:
    const url = `https://n8n02.costamolina.tech/webhook/pipedrive?phone=${phone}`;
    const res = await fetch(url);
    if (!res.ok) {
      throw new Error(`Erro HTTP ${res.status}`);
    }
    const data = await res.json();
    dealInfo.value = data;
  } catch (e) {
    error.value = e.message || 'Falha ao buscar dados do Pipedrive';
  }
});

async function mudarEstagio() {
  if (!dealInfo.value || !dealInfo.value.id) return;
  try {
    // Exemplo de POST p/ atualizar estágio via n8n
    const res = await fetch('https://n8n02.costamolina.tech/webhook/pipedrive-update', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        dealId: dealInfo.value.id,
        novoEstagio: 'Lead Quente',
      }),
    });
    if (!res.ok) {
      throw new Error(`Erro ao atualizar estágio: ${res.status}`);
    }
    const updatedData = await res.json();
    dealInfo.value = updatedData; // Ex: recarregue com dados atualizados
  } catch (e) {
    error.value = e.message;
  }
}
</script>
