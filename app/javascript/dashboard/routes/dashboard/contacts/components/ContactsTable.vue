<script>
import { mapGetters } from 'vuex';
import { VeTable } from 'vue-easytable';

import Spinner from 'shared/components/Spinner.vue';
import Thumbnail from 'dashboard/components/widgets/Thumbnail.vue';
import EmptyState from 'dashboard/components/widgets/EmptyState.vue';
import { dynamicTime } from 'shared/helpers/timeHelper';

export default {
  components: {
    EmptyState,
    Spinner,
    VeTable,
  },
  props: {
    contacts: {
      type: Array,
      default: () => [],
    },
    showSearchEmptyState: {
      type: Boolean,
      default: false,
    },
    onClickContact: {
      type: Function,
      default: () => {},
    },
    isLoading: {
      type: Boolean,
      default: false,
    },
    sortParam: {
      type: String,
      default: 'last_activity_at',
    },
    sortOrder: {
      type: String,
      default: 'desc',
    },
  },
  data() {
    return {
      sortConfig: {},
      sortOption: {
        sortAlways: true,
        sortChange: params => this.$emit('onSortChange', params),
      },
    };
  },
  computed: {
    ...mapGetters({
      isRTL: 'accounts/isRTL',
    }),
    tableData() {
      if (this.isLoading) {
        return [];
      }
      return this.contacts.map(item => {
        // Note: The attributes used here is in snake case
        // as it simplier the sort attribute calculation
        const { last_activity_at: lastActivityAt } = item;
        const { created_at: createdAt } = item;
        return {
          ...item,
          name: item.name || '---',
          conversationsCount: item.conversations_count || '---',
          last_activity_at: lastActivityAt
            ? dynamicTime(lastActivityAt)
            : '---',
          created_at: createdAt ? dynamicTime(createdAt) : '---',
        };
      });
    },
    columns() {
      return [
        {
          field: 'name',
          key: 'name',
          title: this.$t('CONTACTS_PAGE.LIST.TABLE_HEADER.NAME'),
          fixed: 'left',
          align: this.isRTL ? 'right' : 'left',
          sortBy: this.sortConfig.name || '',
          width: 300,
          renderBodyCell: ({ row }) => (
            <woot-button
              variant="clear"
              onClick={() => this.onClickContact(row.id)}
            >
              <div class="row--user-block">
                <Thumbnail
                  src={row.thumbnail}
                  size="32px"
                  username={row.name}
                  status={row.availability_status}
                />
                <div class="user-block">
                  <h6 class="overflow-hidden text-base whitespace-nowrap text-ellipsis">
                    <router-link
                      to={`/app/accounts/${this.$route.params.accountId}/contacts/${row.id}`}
                      class="user-name"
                    >
                      {row.name}
                    </router-link>
                  </h6>
                  <button class="button clear small link view-details--button">
                    {this.$t('CONTACTS_PAGE.LIST.VIEW_DETAILS')}
                  </button>
                </div>
              </div>
            </woot-button>
          ),
        },
        {
          field: 'last_activity_at',
          key: 'last_activity_at',
          sortBy: this.sortConfig.last_activity_at || '',
          title: this.$t('CONTACTS_PAGE.LIST.TABLE_HEADER.LAST_ACTIVITY'),
          align: this.isRTL ? 'right' : 'left',
        },
        {
          field: 'created_at',
          key: 'created_at',
          sortBy: this.sortConfig.created_at || '',
          title: this.$t('CONTACTS_PAGE.LIST.TABLE_HEADER.CREATED_AT'),
          align: this.isRTL ? 'right' : 'left',
        },
      ];
    },
  },
  watch: {
    sortOrder() {
      this.setSortConfig();
    },
    sortParam() {
      this.setSortConfig();
    },
  },
  mounted() {
    this.setSortConfig();
  },
  methods: {
    setSortConfig() {
      this.sortConfig = { [this.sortParam]: this.sortOrder };
    },
  },
};
</script>

<template>
  <section
    class="flex-1 h-full -mt-1 overflow-hidden bg-white contacts-table-wrap dark:bg-slate-900"
  >
    <VeTable
      fixed-header
      max-height="calc(100vh - 7.125rem)"
      scroll-width="187rem"
      :columns="columns"
      :table-data="tableData"
      :border-around="false"
      :sort-option="sortOption"
    />

    <EmptyState
      v-if="showSearchEmptyState"
      :title="$t('CONTACTS_PAGE.LIST.404')"
    />
    <EmptyState
      v-else-if="!isLoading && !contacts.length"
      :title="$t('CONTACTS_PAGE.LIST.NO_CONTACTS')"
    />
    <div v-if="isLoading" class="flex items-center justify-center text-base">
      <Spinner />
      <span>{{ $t('CONTACTS_PAGE.LIST.LOADING_MESSAGE') }}</span>
    </div>
  </section>
</template>

<style lang="scss" scoped>
.contacts-table-wrap::v-deep {
  .ve-table {
    @apply pb-8;
  }
  .row--user-block {
    @apply items-center flex text-left;

    .user-block {
      @apply items-start flex flex-col my-0 mx-2;
    }

    .user-name {
      @apply text-sm font-medium m-0 capitalize;
    }

    .view-details--button {
      @apply text-slate-600 dark:text-slate-200;
    }

    .user-email {
      @apply m-0;
    }
  }

  .ve-table-header-th {
    padding: var(--space-small) var(--space-two) !important;
  }

  .ve-table-body-td {
    padding: var(--space-small) var(--space-two) !important;
  }

  .ve-table-header-th {
    font-size: var(--font-size-mini) !important;
  }
  .ve-table-sort {
    @apply -top-1;
  }
}

.cell--social-profiles {
  a {
    @apply text-slate-300 dark:text-slate-400 text-lg min-w-[2rem] text-center;
  }
}
</style>
