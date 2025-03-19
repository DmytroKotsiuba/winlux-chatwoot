<script>
import PreChatForm from '../components/PreChat/Form.vue';
import configMixin from '../mixins/configMixin';
import routerMixin from '../mixins/routerMixin';
import { isEmptyObject } from 'widget/helpers/utils';
import { ON_CONVERSATION_CREATED } from '../constants/widgetBusEvents';

export default {
  components: {
    PreChatForm,
  },
  mixins: [configMixin, routerMixin],
  mounted() {
    this.$emitter.on(ON_CONVERSATION_CREATED, () => {
      // Redirect to messages page after conversation is created
      this.replaceRoute('messages');
    });
  },
  methods: {
    onSubmit({
      fullName,
      message,
      activeCampaignId,
      contactCustomAttributes,
      conversationCustomAttributes,
    }) {
      if (activeCampaignId) {
        this.$emitter.emit('execute-campaign', {
          campaignId: activeCampaignId,
          customAttributes: conversationCustomAttributes,
        });
        this.$store.dispatch('contacts/update', {
          user: {
            name: fullName,
          },
        });
      } else {
        this.$store.dispatch('conversation/createConversation', {
          fullName: fullName,
          message: message,
          customAttributes: conversationCustomAttributes,
        });
      }
      if (!isEmptyObject(contactCustomAttributes)) {
        this.$store.dispatch(
          'contacts/setCustomAttributes',
          contactCustomAttributes
        );
      }
    },
  },
};
</script>

<template>
  <div class="flex flex-1 overflow-auto">
    <PreChatForm :options="preChatFormOptions" @submit="onSubmit" />
  </div>
</template>
