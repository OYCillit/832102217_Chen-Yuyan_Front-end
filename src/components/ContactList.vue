<script setup>
import { ref, onMounted, computed } from 'vue'

const contacts = ref([])
const newContact = ref({ name: '', phone: '' })
const editingContact = ref(null)
const searchQuery = ref('')

// 获取所有联系人
const fetchContacts = async () => {
  try {
    const response = await fetch('http://localhost:3000/contacts')
    contacts.value = await response.json()
  } catch (error) {
    console.error('获取联系人失败:', error)
  }
}

// 添加联系人
const addContact = async () => {
  if (!newContact.value.name || !newContact.value.phone) {
    alert('请填写完整的联系人信息')
    return
  }
  
  try {
    const response = await fetch('http://localhost:3000/contacts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newContact.value)
    })
    if (response.ok) {
      await fetchContacts()
      newContact.value = { name: '', phone: '' }
    }
  } catch (error) {
    console.error('添加联系人失败:', error)
  }
}

// 删除联系人
const deleteContact = async (id) => {
  if (!confirm('确定要删除这个联系人吗？')) return
  
  try {
    const response = await fetch(`http://localhost:3000/contacts/${id}`, {
      method: 'DELETE'
    })
    if (response.ok) {
      await fetchContacts()
    }
  } catch (error) {
    console.error('删除联系人失败:', error)
  }
}

// 开始编辑联系人
const startEdit = (contact) => {
  editingContact.value = { ...contact }
}

// 保存编辑的联系人
const saveEdit = async () => {
  if (!editingContact.value.name || !editingContact.value.phone) {
    alert('请填写完整的联系人信息')
    return
  }

  try {
    const response = await fetch(`http://localhost:3000/contacts/${editingContact.value.id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editingContact.value)
    })
    if (response.ok) {
      await fetchContacts()
      editingContact.value = null
    }
  } catch (error) {
    console.error('更新联系人失败:', error)
  }
}

// 过滤联系人列表
const filteredContacts = computed(() => {
  return contacts.value.filter(contact => 
    contact.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    contact.phone.includes(searchQuery.value)
  )
})

onMounted(fetchContacts)
</script>

<template>
  <div class="contact-container">
    <!-- 左侧添加联系人面板 -->
    <div class="add-contact-panel">
      <div class="add-contact-header">
        <i class="bi bi-person-plus"></i>
        添加新联系人
      </div>
      <div class="add-contact-form">
        <div class="form-group">
          <label>姓名</label>
          <input 
            type="text" 
            class="form-control" 
            v-model="newContact.name" 
            placeholder="请输入姓名"
          >
        </div>
        <div class="form-group">
          <label>电话号码</label>
          <input 
            type="tel" 
            class="form-control" 
            v-model="newContact.phone" 
            placeholder="请输入电话号码"
          >
        </div>
        <button 
          class="btn btn-primary w-100" 
          @click="addContact"
        >
          添加联系人
        </button>
      </div>
    </div>

    <!-- 右侧联系人列表 -->
    <div class="contacts-panel">
      <div class="contacts-header">
        <div class="search-box">
          <i class="bi bi-search"></i>
          <input 
            type="text" 
            class="form-control" 
            v-model="searchQuery"
            placeholder="搜索联系人..."
          >
        </div>
      </div>

      <div class="contacts-list">
        <div class="list-header">
          <span class="name-col">姓名</span>
          <span class="phone-col">电话号码</span>
          <span class="action-col">操作</span>
        </div>

        <div class="list-content">
          <div v-for="contact in filteredContacts" 
               :key="contact.id" 
               class="contact-item"
          >
            <!-- 显示模式 -->
            <template v-if="editingContact?.id !== contact.id">
              <div class="name-col">
                <i class="bi bi-person"></i>
                <span class="name-text">{{ contact.name }}</span>
              </div>
              <div class="phone-col">
                <i class="bi bi-telephone"></i>
                <span class="phone-number">{{ contact.phone }}</span>
              </div>
              <div class="action-col">
                <button class="btn btn-link" @click="startEdit(contact)">
                  <i class="bi bi-pencil text-primary"></i>
                </button>
                <button class="btn btn-link" @click="deleteContact(contact.id)">
                  <i class="bi bi-trash text-danger"></i>
                </button>
              </div>
            </template>

            <!-- 编辑模式 -->
            <template v-else>
              <div class="edit-mode">
                <input 
                  type="text" 
                  class="form-control" 
                  v-model="editingContact.name"
                >
                <input 
                  type="tel" 
                  class="form-control" 
                  v-model="editingContact.phone"
                >
                <div class="edit-actions">
                  <button class="btn btn-link" @click="saveEdit">
                    <i class="bi bi-check-lg text-success"></i>
                  </button>
                  <button class="btn btn-link" @click="editingContact = null">
                    <i class="bi bi-x-lg text-secondary"></i>
                  </button>
                </div>
              </div>
            </template>
          </div>

          <!-- 空状态显示 -->
          <div v-if="filteredContacts.length === 0" class="empty-state">
            暂无联系人
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.contact-container {
  display: flex;
  min-height: 100vh;
  background-color: #fff;
  max-width: 1920px; /* 限制最大宽度 */
  margin: 0 auto; /* 居中显示 */
}

.add-contact-panel {
  width: 300px;
  min-width: 250px; /* 最小宽度 */
  border-right: 1px solid #eee;
  padding: 20px;
  height: 100vh;
  position: sticky;
  top: 0;
  overflow-y: auto;
}

.add-contact-header {
  font-size: clamp(16px, 2vw, 18px); /* 响应式字体大小 */
  font-weight: bold;
  color: #1a73e8;
  margin-bottom: 20px;
}

.add-contact-header i {
  margin-right: 8px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #666;
}

.contacts-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0; /* 防止内容溢出 */
}

.contacts-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
  position: sticky;
  top: 0;
  background: white;
  z-index: 1;
}

.search-box {
  position: relative;
  max-width: min(300px, 100%); /* 响应式宽度 */
}

.search-box i {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: #666;
}

.search-box input {
  padding-left: 35px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 100%;
}

.contacts-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 0 20px;
}

.list-header {
  display: flex;
  padding: 15px 0;
  border-bottom: 1px solid #eee;
  font-weight: 500;
  color: #666;
  position: sticky;
  top: 80px; /* contacts-header高度 + padding */
  background: white;
  z-index: 1;
}

.list-content {
  flex: 1;
  overflow-y: auto;
}

.contact-item {
  display: flex;
  padding: 12px 0;
  border-bottom: 1px solid #eee;
  align-items: center;
}

.name-col {
  flex: 2;
  min-width: 120px; /* 最小宽度 */
  padding-right: 10px;
}

.phone-col {
  flex: 2;
  min-width: 120px;
  padding-right: 10px;
}

.action-col {
  flex: 1;
  min-width: 80px;
  text-align: right;
}

.contact-item i {
  margin-right: 8px;
  color: #666;
}

.edit-mode {
  display: flex;
  width: 100%;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap; /* 在小屏幕上换行 */
}

.edit-mode input {
  flex: 1;
  min-width: 120px;
}

.edit-actions {
  display: flex;
  gap: 5px;
}

.btn-link {
  padding: 4px 8px;
  color: inherit;
  text-decoration: none;
}

.empty-state {
  text-align: center;
  padding: 40px;
  color: #666;
}

.phone-number {
  color: #333; /* 更深的文字颜色 */
  font-weight: 500; /* 稍微加粗 */
}

.name-text,
.phone-number {
  color: #333;
  font-weight: 500;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .contact-container {
    flex-direction: column;
  }

  .add-contact-panel {
    width: 100%;
    height: auto;
    border-right: none;
    border-bottom: 1px solid #eee;
    position: relative;
  }

  .contacts-panel {
    height: calc(100vh - 250px); /* 减去添加联系人面板的高度 */
  }

  .list-header {
    top: 70px;
  }

  .contact-item {
    flex-wrap: wrap;
  }

  .name-col, .phone-col {
    flex: 1 1 40%;
  }

  .action-col {
    flex: 1 1 100%;
    text-align: left;
    margin-top: 10px;
  }
}

/* 超大屏幕优化 */
@media (min-width: 1920px) {
  .contact-container {
    border-left: 1px solid #eee;
    border-right: 1px solid #eee;
  }
}

/* 小屏幕优化 */
@media (max-width: 480px) {
  .edit-mode {
    flex-direction: column;
    align-items: stretch;
  }

  .edit-mode input {
    width: 100%;
  }

  .edit-actions {
    justify-content: flex-end;
    margin-top: 10px;
  }
}
</style> 