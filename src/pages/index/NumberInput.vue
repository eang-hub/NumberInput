<template>
  <br>
  internalValue:{{internalValue}}
  <wd-input
      type="text"
      v-model="internalValue"
      v-bind="$attrs"
      @input="handleInput"
      @focus="handleFocus"
      @blur="handleBlur"
      @clear="handleClear"
      :placeholder="props.placeholder"
  />
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

interface Props {
  modelValue?: string | number | undefined
  max?: number
  min?: number
  precision?: number
  thousandSeparator?: boolean
  placeholder?: string
}

interface InputEvent {
  value: string
  cursor: number
  keyCode: number
}

const props = withDefaults(defineProps<Props>(), {
  max: 9999999999,
  min: 0,
  precision: 2,
  thousandSeparator: true,
  placeholder: "请输入数值"
})

const emit = defineEmits<{
  'update:modelValue': [value: string | number | undefined]
  'input': [event: InputEvent]
  'focus': [event: Event]
  'blur': [event: Event]
  'clear': []
}>()

const internalValue = ref<string>('')

/** 格式化数字（加千分位） */
const formatNumber = (value: number): string => {
  if (!props.thousandSeparator) return value.toString()
  const parts = value.toString().split('.')
  parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',')
  return parts.join('.')
}

/** 去掉千分位 */
const removeThousandSeparator = (value: string): string => {
  return value.replace(/,/g, '')
}

/** 限制小数位数 */
const limitPrecision = (value: string): string => {
  if (props.precision === 0) {
    return value.replace(/\./g, '')
  }
  const parts = value.split('.')
  if (parts.length > 2) {
    parts[1] = parts.slice(1).join('')
  }
  if (parts[1] && parts[1].length > props.precision) {
    parts[1] = parts[1].substring(0, props.precision)
  }
  return parts.length > 1 ? `${parts[0]}.${parts[1]}` : parts[0]
}

/** 范围校验 */
const validateRange = (value: number): number => {
  if (value > props.max) return props.max
  if (value < props.min) return props.min
  return value
}

/** 输入事件 */
const handleInput = (event: InputEvent) => {
  let inputValue = event.value

  // 只允许数字、小数点和负号
  inputValue = inputValue.replace(/[^\d.-]/g, '')

  // 限制小数位数
  inputValue = limitPrecision(inputValue)

  // 更新内部值
  internalValue.value = inputValue

  // 处理 emit
  if (!inputValue || inputValue === '' || inputValue === '-') {
    emit('update:modelValue', '')
  } else {
    const cleanValue = removeThousandSeparator(inputValue)
    const numValue = parseFloat(cleanValue)

    if (isNaN(numValue)) {
      internalValue.value = ''
      emit('update:modelValue', '')
    } else {
      const validatedValue = validateRange(numValue)
      emit('update:modelValue', validatedValue)
    }
  }

  emit('input', event)
}

/** 聚焦 */
const handleFocus = (event: Event) => {
  emit('focus', event)
}

/** 失焦 */
const handleBlur = (event: Event) => {
  if (internalValue.value) {
    const cleanValue = removeThousandSeparator(internalValue.value)
    const numValue = parseFloat(cleanValue)

    if (!isNaN(numValue)) {
      const validatedValue = validateRange(numValue)
      internalValue.value = props.thousandSeparator
          ? formatNumber(validatedValue)
          : validatedValue.toString()
      emit('update:modelValue', validatedValue)
    } else {
      internalValue.value = ''
      emit('update:modelValue', '')
    }
  }
  emit('blur', event)
}

/** 清空 */
const handleClear = () => {
  internalValue.value = ''
  emit('update:modelValue', '')
  emit('clear')
}

/** 监听外部传入的值 */
watch(
    () => props.modelValue,
    (newValue) => {
      if (newValue === undefined || newValue === null || newValue === '') {
        internalValue.value = ''
      } else {
        internalValue.value = props.thousandSeparator
            ? formatNumber(Number(newValue))
            : newValue.toString()
      }
    },
    { immediate: true }
)
</script>

<script lang="ts">
export default {
  name: 'NumberInput',
  inheritAttrs: false
}
</script>
