<template>
  <div>
    <a-tooltip>
        <template #title>
            <a-button title="Copy Coupon Code" @click="copyCouponCode('coupon-code', record.id)">
                Copy
            </a-button>
        </template>
        <a-tag color="#f50">
            {{ record.couponCode }}
            <a-input :id="`coupon-code${record.id}`" :value="record.couponCode" style="width: 0; height: 0; margin: 0; padding: 0; visibility: hidden"/>
        </a-tag>
    </a-tooltip>
  </div>
</template>

<script setup>
import {message} from "ant-design-vue";
 /* Function to copy */  
const copyCouponCode = (key, id) => {
    const idSelector = `#${key}${id}`;
    console.log('id', idSelector);
    let testingCodeToCopy = document.querySelector(`${idSelector}`);
    console.log('testingCodeToCopy', testingCodeToCopy);
    (async() => {
        await testingCodeToCopy
        console.table('table', testingCodeToCopy)
        testingCodeToCopy?.setAttribute('type', 'text');
        console.log('setAttribute', testingCodeToCopy.setAttribute('type', 'text'))
        testingCodeToCopy?.select();
        console.log('select', testingCodeToCopy.select())
        try {
            const successful = document.execCommand('copy');
            const msg = successful ? 'successful' : 'unsuccessful';
            message.success({
                content: () => 'Successfully copy',
                style: {
                    marginTop: '5vh',
                },
            });
        } catch (err) {
            message.error({
                content: () => 'Oops, unable to copy',
                style: {
                    marginTop: '5vh',
                },
            });
        }
        /* unselect the range */
        window.getSelection().removeAllRanges();
    })()
}; /* Function to copy */
</script>
