<template>
  <div class="customer-tree">
    <a-tree
      :tree-data="treeData"
      show-icon
      default-expand-all
    >
        <template #icon="{ key }">
            <template v-if="key.split('-').length === 2">
                <StarOutlined title="Họ Tên"/>
            </template>
            <template v-else-if="key.split('-').length === 3 && key[key.length-1] === '0'">
                <MailOutlined title="Email" />
            </template>
            <template v-else-if="key.split('-').length === 3 && key[key.length-1] === '1'">
                <MobileOutlined title="Điện thoại" />
            </template>
            <template v-else-if="key.split('-').length === 3 && key[key.length-1] === '2'">
                <WhatsAppOutlined title="WhatsApp" />
            </template>
            <template v-else-if="key.split('-').length === 3 && key[key.length-1] === '3'">
                <IdcardOutlined title="Chức vụ" />
            </template>
            <template v-else-if="key.split('-').length === 3 && key[key.length-1] === '4'">
                <BorderlessTableOutlined title="Mã bưu chính" />
            </template>
        </template>
    </a-tree>
  </div>
</template>

<script setup>
import {
    StarOutlined,
    MailOutlined,
    MobileOutlined,
    WhatsAppOutlined,
    IdcardOutlined,
    BorderlessTableOutlined,
} from '@ant-design/icons-vue';

const props = defineProps({
  dataCustomers: {
    type: Array,
    default: [],
    re: [],
  },
});

const customerData = props.dataCustomers;
const treeData = customerData?.map((customer, index) => ({
    title: customer.fullName || customer.name,
    key: `0-${index}`,
    children: [
        {
            title: `${customer.email}`,
            key: `0-${index}-0`,
        },
        {
            title: `${customer.phone}`,
            key: `0-${index}-1`,
        },
        {
            title: `${customer.whatsapp}`,
            key: `0-${index}-2`,
        },
        {
            title: `${customer.position}`,
            key: `0-${index}-3`,
        },
        {
            title: `${customer.postalCode}`,
            key: `0-${index}-4`,
        },
    ],
}));
</script>
