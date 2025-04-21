<template>
  <div class="email-input-container" @click="focusInput">
    <div class="tags-input">
      <span 
        v-for="(email, index) in emailTags" 
        :key="index" 
        :class="['email-tag', email.valid ? 'valid' : 'invalid']"
      >
        {{ email.address }}
        <span class="remove-tag" @click.stop="removeTag(index)">×</span>
      </span>
      <input
        ref="emailInput"
        v-model="currentInput"
        type="text"
        placeholder="输入收件人邮箱，多个用逗号、分号、空格或回车分隔"
        @keydown="handleKeyDown"
        @blur="handleBlur"
        @paste="handlePaste"
      />
    </div>
  </div>
  <div v-if="errorMessage" class="error-message">
    {{ errorMessage }}
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const emailInput = ref(null)
const currentInput = ref('')
const emailTags = ref([])
const errorMessage = ref('')

// 邮箱校验正则
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

// 处理键盘事件
const handleKeyDown = (e) => {
  // 回车、逗号或分号键触发添加邮箱
  if (e.key === 'Enter' || e.key === ',' || e.key === '，' || e.key === ';' || e.key === '；' || e.key === ' ') {
    e.preventDefault()
    addEmails()
  }
  
  // 处理退格键
  if (e.key === 'Backspace' && currentInput.value === '' && emailTags.value.length > 0) {
    emailTags.value.pop()
  }
}

// 添加邮箱
const addEmails = () => {
  errorMessage.value = ''
  if (!currentInput.value.trim()) return
  
  // 分割多个邮箱（支持逗号、分号分隔）
  const newEmails = currentInput.value.split(/[,;]\s*/)
  let hasError = false
  
  newEmails.forEach(email => {
    email = email.trim()
    if (!email) return
    
    const isValid = emailRegex.test(email)
    const isDuplicate = emailTags.value.some(tag => tag.address === email)
    
    if (isDuplicate) {
      errorMessage.value = `邮箱 ${email} 已存在`
      hasError = true
      return
    }
    
    emailTags.value.push({
      address: email,
      valid: isValid
    })
    
    if (!isValid) {
      errorMessage.value = `邮箱 ${email} 格式不正确`
      hasError = true
    }
  })
  
  currentInput.value = ''
  if (!hasError) errorMessage.value = ''
  
  // 保持焦点
  nextTick(() => {
    emailInput.value.focus()
  })
}

// 处理失焦事件
const handleBlur = () => {
  addEmails()
}

// 处理粘贴
const handlePaste = (e) => {
  setTimeout(() => {
    addEmails()
  }, 0)
}

// 删除邮箱标签
const removeTag = (index) => {
  emailTags.value.splice(index, 1)
  nextTick(() => {
    emailInput.value.focus()
  })
}

// 点击容器聚焦输入框
const focusInput = () => {
  emailInput.value.focus()
}

// 暴露方法
defineExpose({
  getEmails: () => emailTags.value.filter(tag => tag.valid).map(tag => tag.address),
  hasInvalidEmails: () => emailTags.value.some(tag => !tag.valid),
  clearEmails: () => {
    emailTags.value = []
    currentInput.value = ''
    errorMessage.value = ''
  }
})
</script>

<style scoped>
.email-input-container {
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  padding: 5px;
  min-height: 40px;
  width: 1000px;
  background: white;
  cursor: text;
}

.tags-input {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 5px;
}

.email-tag {
  display: inline-flex;
  align-items: center;
  padding: 0 10px;
  height: 28px;
  line-height: 28px;
  font-size: 14px;
  border-radius: 4px;
  box-sizing: border-box;
  white-space: nowrap;
  background-color: #f0f0f0;
  border: 1px solid #d9d9d9;
}

.email-tag.valid {
  background-color: #e6f7ff;
  color: #1890ff;
  border: 1px solid #1890ff;
}

.email-tag.invalid {
  background-color: #fff1f0;
  color: #ff4d4f;
  border: 1px solid #ff4d4f;
}

.remove-tag {
  margin-left: 5px;
  cursor: pointer;
  font-size: 16px;
  color: inherit;
}

input {
  flex: 1;
  min-width: 100px;
  height: 32px;
  line-height: 32px;
  padding: 0 10px;
  font-size: 14px;
  border: none;
  outline: none;
  box-sizing: border-box;
}

.error-message {
  color: #ff4d4f;
  font-size: 12px;
  margin-top: 5px;
  padding-left: 5px;
}
</style>