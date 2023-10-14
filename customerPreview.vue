<template>
  <div class="customers-preview">
    <a-space>
      <template v-for="(customer, index) in props.dataCustomers">
        <a-popover overlayClassName="customers-preview-overlay" :placement="props.placement" :auto-adjust-overflow="false">
            <temlate v-if="customer?.fullName || customer.name">
                <a-tag :color="colorStyle[index%4]" class="customers-full-name">
                    {{ customer.fullName ? customer.fullName : customer.name }}
                </a-tag>
            </temlate>
            <temlate v-else>
                <span class="info-not-updated-yet"> Họ tên chưa cập nhật </span>
            </temlate>
            <template #content>
            <div class="sub-customer">
              <div class="avatar">
                <a-avatar v-if="customer.avatar != null" :src="customer.avatar" :size="100" />
                <a-avatar v-else :style="`background-color: #${colorStyle[index]}`" :size="100">
                  <strong>{{ customer.firstName ? customer.firstName.charAt(0).toUpperCase() : 'N' }}</strong>
                </a-avatar>
              </div>
              <p class="fullName">
                <strong>
                  <span v-if="customer.fullName || customer.name">
                    {{ customer.fullName || customer.name}}
                  </span>
                  <span v-else class="info-not-updated-yet"> Họ tên chưa cập nhật </span>
                </strong>
              </p>
              <p class="email">
                <span class="icon"><mail-outlined /></span>
                <span style="display: inline-block; width: 50px">Email:</span>
                <span v-if="customer.email">
                  <a :href="`mailto:${customer.email}`">{{ customer.email }}</a>
                </span>
                <span v-else class="info-not-updated-yet"> Chưa cập nhật </span>
              </p>
              <p class="phone">
                <span class="icon"><mobile-outlined /></span>
                <span style="display: inline-block; width: 50px">Phone: </span>
                <span v-if="customer.phone">
                  <a :href="`tel:${customer.phone}`">{{ customer.phone }}</a>
                </span>
                <span v-else class="info-not-updated-yet"> Chưa cập nhật </span>
              </p>
              <div class="button-view">
                <NuxtLink :to="`/crm/customer/${customer.id}/detail`">Thông tin chi tiết</NuxtLink>
              </div>
            </div>
        </template>
        </a-popover>
      </template>
    </a-space>
  </div>
</template>

<script setup>
import { MailOutlined, MobileOutlined } from '@ant-design/icons-vue';
import { watchEffect } from 'vue';

const colorStyle = ['#f50', '#2db7f5', '#87d068', '#108ee9'];

const props = defineProps({
  dataCustomers: {
    type: Array,
    default: [],
  },
  placement: {
      type: String,
      default: 'top',
  },
});
</script>

<style lang="scss">
.customers-preview {
  .info-not-updated-yet {
    font-style: italic;
    color: #ccc;
  }
}
.customers-preview-overlay {
  position: absolute;
  z-index: 10000;
  padding-top: 10%;
  padding-right: 32%;
  .info-not-updated-yet {
    font-style: italic;
    color: #ccc;
  }
  .ant-popover-inner-content {
    padding: 0;
  }
  .sub-customer {
    width: 300px;
    position: absolute;
    bottom: 0;
    left: 0;
    background: #fff;
    border-radius: 18px;
    padding: 20px;
    border: 1px solid rgba(0, 0, 0, 0.01);
    box-shadow: 0 1px 10px #00000040;
    .avatar {
      margin-bottom: 20px;
    }
    .name {
      font-size: 20px;
    }
    .icon {
      width: 30px;
      display: inline-block;
    }
    .button-view {
      a {
        padding: 10px 0;
        border-radius: 10px;
        border: 1px solid #e1e1e1;
        display: block;
        text-align: center;
        color: #000;
        font-weight: 500;
      }
    }
    .ant-avatar-string {
      font-size: 5rem;
    }
  }
}
</style>
