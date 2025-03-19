<script>
import { useAlert } from 'dashboard/composables';
import { ExceptionWithMessage } from 'shared/helpers/CustomErrors';
import { useVuelidate } from '@vuelidate/core';

export default {
  props: {
    contact: {
      type: Object,
      default: () => ({}),
    },
    inProgress: {
      type: Boolean,
      default: false,
    },
    onSubmit: {
      type: Function,
      default: () => {},
    },
  },
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      description: '',
      avatarFile: null,
      avatarUrl: '',
    };
  },
  validations: {
    description: {},
  },
  watch: {
    contact() {
      this.setContactObject();
    },
  },
  mounted() {
    this.setContactObject();
  },
  methods: {
    onCancel() {
      this.$emit('cancel');
    },
    onSuccess() {
      this.$emit('success');
    },
    setContactObject() {
      const additionalAttributes = this.contact.additional_attributes || {};
      this.description = additionalAttributes.description || '';
      this.avatarUrl = this.contact.thumbnail || '';
    },
    getContactObject() {
      const contactObject = {
        id: this.contact.id,
        additional_attributes: {
          ...this.contact.additional_attributes,
          description: this.description,
        },
      };
      if (this.avatarFile) {
        contactObject.avatar = this.avatarFile;
        contactObject.isFormData = true;
      }
      return contactObject;
    },
    async handleSubmit() {
      this.v$.$touch();
      if (this.v$.$invalid) {
        return;
      }
      try {
        await this.onSubmit(this.getContactObject());
        this.onSuccess();
        useAlert(this.$t('CONTACT_FORM.SUCCESS_MESSAGE'));
      } catch (error) {
        if (error instanceof ExceptionWithMessage) {
          useAlert(error.data);
        } else {
          useAlert(this.$t('CONTACT_FORM.ERROR_MESSAGE'));
        }
      }
    },
    handleImageUpload({ file, url }) {
      this.avatarFile = file;
      this.avatarUrl = url;
    },
    async handleAvatarDelete() {
      try {
        if (this.contact && this.contact.id) {
          await this.$store.dispatch('contacts/deleteAvatar', this.contact.id);
          useAlert(this.$t('CONTACT_FORM.DELETE_AVATAR.API.SUCCESS_MESSAGE'));
        }
        this.avatarFile = null;
        this.avatarUrl = '';
      } catch (error) {
        useAlert(
          error.message
            ? error.message
            : this.$t('CONTACT_FORM.DELETE_AVATAR.API.ERROR_MESSAGE')
        );
      }
    },
  },
};
</script>

<template>
  <form
    class="w-full px-8 pt-6 pb-8 contact--form"
    @submit.prevent="handleSubmit"
  >
    <div>
      <div class="w-full">
        <woot-avatar-uploader
          :label="$t('CONTACT_FORM.FORM.AVATAR.LABEL')"
          :src="avatarUrl"
          :delete-avatar="!!avatarUrl"
          class="settings-item"
          @change="handleImageUpload"
          @onAvatarDelete="handleAvatarDelete"
        />
      </div>
    </div>
    <div class="w-full">
      <label :class="{ error: v$.description.$error }">
        {{ $t('CONTACT_FORM.FORM.BIO.LABEL') }}
        <textarea
          v-model.trim="description"
          type="text"
          :placeholder="$t('CONTACT_FORM.FORM.BIO.PLACEHOLDER')"
          @input="v$.description.$touch"
        />
      </label>
    </div>
    <div class="flex flex-row justify-end w-full gap-2 px-0 py-2">
      <div class="w-full">
        <woot-submit-button
          :loading="inProgress"
          :button-text="$t('CONTACT_FORM.FORM.SUBMIT')"
        />
        <button class="button clear" @click.prevent="onCancel">
          {{ $t('CONTACT_FORM.FORM.CANCEL') }}
        </button>
      </div>
    </div>
  </form>
</template>

<style scoped lang="scss">
::v-deep {
  .multiselect .multiselect__tags .multiselect__single {
    @apply pl-0;
  }
}
</style>
