<template>
  <transition name="fade">
    <SfTabs
      v-if="edittingAddress"
      key="edit-address"
      :open-tab="1"
      class="tab-orphan"
    >
      <SfTab :title="isNewAddress ? 'Add the address' : 'Update the address'">
        <p class="message">
          {{ $t('Contact details updated') }}
        </p>

        <ShippingAddressForm
          :cancel="cancelChangeAddress"
          :address="activeAddress"
          :isNew="isNewAddress"
          :onSubmit="saveAddress"
        />
      </SfTab>
    </SfTabs>

    <SfTabs v-else :open-tab="1" key="address-list" class="tab-orphan">
      <SfTab title="Shipping details">
        <p class="message">
          {{ $t('Manage shipping addresses') }}
        </p>
        <SfLoader :class="{ loader: loading }" :loading="loading">
          <transition-group tag="div" name="fade" class="shipping-list">
            <div
              v-for="address in addresses"
              :key="address.id"
              class="shipping"
            >
              <div class="shipping__content">
                <div class="shipping__address">
                  <UserShippingAddress :address="address" />
                </div>
              </div>
              <div class="shipping__actions">
                <SfIcon
                  icon="cross"
                  color="gray"
                  size="14px"
                  role="button"
                  class="smartphone-only"
                  @click="removeAddress(address)"
                />
                <SfButton @click="changeAddress(address)">
                  {{ $t('Change') }}
                </SfButton>

                <SfButton
                  class="color-light shipping__button-delete desktop-only"
                  @click="removeAddress(address)"
                >
                  {{ $t('Delete') }}
                </SfButton>
              </div>
            </div>
          </transition-group>
        </SfLoader>

        <SfButton class="action-button" @click="changeAddress()">
          {{ $t('Add new address') }}
        </SfButton>
      </SfTab>
    </SfTabs>
  </transition>
</template>

<script lang="ts">
import {
  ref,
  defineComponent,
  computed,
  onMounted
} from '@nuxtjs/composition-api';
import { storeToRefs } from 'pinia';
import { UserAddress } from '@vsf-enterprise/bigcommerce-api';
import { SfTabs, SfButton, SfIcon, SfLoader } from '@storefront-ui/vue';
import UserShippingAddress from '~/components/UserShippingAddress.vue';
import ShippingAddressForm from '~/components/MyAccount/ShippingAddressForm.vue';
import { useUserShipping } from '~/composables/useUserShipping';
import { getAddresses } from '~/composables/useUserShipping/helpers';
import { useCustomerStore } from '~/stores/customer';

export default defineComponent({
  name: 'ShippingDetails',
  components: {
    SfTabs,
    SfButton,
    SfLoader,
    SfIcon,
    UserShippingAddress,
    ShippingAddressForm
  },
  setup() {
    const {
      load: loadUserShipping,
      loading,
      addAddress,
      deleteAddress,
      updateAddress
    } = useUserShipping();
    const customerStore = useCustomerStore();
    const { shipping } = storeToRefs(customerStore);
    const addresses = computed(() => getAddresses(shipping.value));
    const edittingAddress = ref(false);
    const activeAddress = ref(undefined);
    const isNewAddress = computed(() => !activeAddress.value);
    const changeAddress = (address = undefined) => {
      activeAddress.value = address;
      edittingAddress.value = true;
    };

    const cancelChangeAddress = () => {
      activeAddress.value = undefined;
      edittingAddress.value = false;
    };

    const removeAddress = async (address: UserAddress) => {
      await deleteAddress(address);
      await loadUserShipping();
    };

    const saveAddress = async ({ form, onComplete, onError }) => {
      try {
        const actionMethod = isNewAddress.value ? addAddress : updateAddress;
        const data = await actionMethod(form);
        edittingAddress.value = false;
        activeAddress.value = undefined;
        await onComplete(data);
        await loadUserShipping();
      } catch (error) {
        onError(error);
      }
    };

    onMounted(async () => {
      await loadUserShipping();
    });

    return {
      changeAddress,
      updateAddress,
      removeAddress,
      saveAddress,
      addresses,
      edittingAddress,
      activeAddress,
      isNewAddress,
      loading,
      cancelChangeAddress
    };
  }
});
</script>

<style lang="scss" scoped>
.message {
  font-family: var(--font-family--primary);
  line-height: 1.6;
  font-size: var(--font-size--base);
  margin: 0 0 var(--spacer-base);
}
.shipping-list {
  margin-bottom: var(--spacer-base);
}
.shipping {
  display: flex;
  padding: var(--spacer-xl) 0;
  border-top: 1px solid var(--c-light);
  &:last-child {
    border-bottom: 1px solid var(--c-light);
  }
  &__content {
    flex: 1;
    color: var(--c-text);
    font-size: var(--font-size--base);
    font-weight: 300;
    line-height: 1.6;
  }
  &__actions {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: flex-end;
    @include for-desktop {
      flex-direction: row;
      align-items: center;
      justify-content: flex-end;
    }
  }
  &__button-delete {
    color: var(--c-link);
    @include for-desktop {
      margin-left: var(--spacer-base);
    }
  }
  &__address {
    margin: 0;
    p {
      margin: 0;
    }
  }
  &__client-name {
    font-size: var(--font-size--base);
    font-weight: 500;
  }
}
.action-button {
  width: 100%;
  @include for-desktop {
    width: auto;
  }
}
.tab-orphan {
  @include for-mobile {
    ::v-deep .sf-tabs {
      &__title {
        display: none;
      }
      &__content {
        border: 0;
        padding: 0;
      }
    }
  }
}
</style>
