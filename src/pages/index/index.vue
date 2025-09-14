<template>
  <view
    class="text-2xl italic flex justify-center items-center h-screen flex-col"
  >
    <text class="hover:(bg-gray-400 font-medium)">Hello UnoCSS！</text>

    value:{{value2}}
    <NumberInput
        key="1"
        v-model="value2"
        label="配置"
    />
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import NumberInput from './NumberInput.vue'

const value2 = ref<number | undefined>()
const value1 = ref<number | undefined>()


</script>
