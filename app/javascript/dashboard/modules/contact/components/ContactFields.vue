<script>
import Attribute from './ContactAttribute.vue';

export default {
  components: { Attribute },
  props: {
    contact: {
      type: Object,
      default: () => ({}),
    },
  },

  computed: {
    additionalAttributes() {
      return this.contact.additional_attributes || {};
    },
    customAttributes() {
      const { custom_attributes: customAttributes = {} } = this.contact;
      return customAttributes;
    },
    customAttributekeys() {
      return Object.keys(this.customAttributes).filter(key => {
        const value = this.customAttributes[key];
        return value !== null && value !== undefined;
      });
    },
  },
  methods: {
    onLocationUpdate(value) {
      this.$emit('update', { location: value });
    },
    handleCustomCreate() {
      this.$emit('createAttribute');
    },
    onCustomAttributeUpdate(key, value) {
      this.$emit('update', { custom_attributes: { [key]: value } });
    },
  },
};
</script>

<template>
  <div class="contact-fields">
    <div
      v-for="attribute in customAttributekeys"
      :key="attribute"
      class="custom-attribute--row"
    >
      <Attribute
        :label="attribute"
        icon="chevron-right"
        :value="customAttributes[attribute]"
        show-edit
        @update="value => onCustomAttributeUpdate(attribute, value)"
      />
    </div>
    <woot-button
      size="small"
      variant="link"
      icon="add"
      @click="handleCustomCreate"
    >
      {{ $t('CUSTOM_ATTRIBUTES.ADD.TITLE') }}
    </woot-button>
  </div>
</template>

<style scoped lang="scss">
.contact-fields {
  margin-top: var(--space-medium);
}

.title {
  margin-bottom: var(--space-normal);
}
</style>
