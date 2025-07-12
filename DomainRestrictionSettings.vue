<template>
  <div class="domain-restriction-settings">
    <div class="settings-header">
      <h3>SSO Domain Restrictions</h3>
      <p class="settings-description">
        Control which domains are allowed to login using SSO providers like Microsoft
      </p>
    </div>

    <div class="auth-provider-section">
      <div class="provider-header">
        <div class="provider-info">
          <span class="provider-icon">🔐</span>
          <span class="provider-name">Microsoft SSO</span>
        </div>
        <span class="provider-status" :class="{ active: isProviderEnabled }">
          {{ isProviderEnabled ? 'Enabled' : 'Disabled' }}
        </span>
      </div>

      <div v-if="isProviderEnabled" class="restriction-controls">
        <div class="radio-group">
          <label class="radio-option">
            <input 
              type="radio" 
              v-model="restrictionMode" 
              value="allow-all"
            />
            <span class="radio-label">Allow all users</span>
            <span class="radio-description">Any user with a valid Microsoft account can login</span>
          </label>

          <label class="radio-option">
            <input 
              type="radio" 
              v-model="restrictionMode" 
              value="domain-restricted"
            />
            <span class="radio-label">Restrict to specific domains</span>
            <span class="radio-description">Only users from specified domains can login</span>
          </label>
        </div>

        <div v-if="restrictionMode === 'domain-restricted'" class="domain-management">
          <div class="domain-input-section">
            <label class="input-label">Allowed domains</label>
            <div class="domain-input-group">
              <input 
                type="text" 
                v-model="newDomain"
                placeholder="e.g. prosegur.com"
                class="domain-input"
                @keyup.enter="addDomain"
              />
              <button 
                @click="addDomain" 
                :disabled="!isValidDomain"
                class="add-button"
              >
                Add Domain
              </button>
            </div>
            <p class="input-hint">Enter domain names without the @ symbol</p>
          </div>

          <div v-if="allowedDomains.length > 0" class="domains-list">
            <div class="list-header">
              <span>{{ allowedDomains.length }} domain(s) configured</span>
            </div>
            <div class="domain-items">
              <div 
                v-for="domain in allowedDomains" 
                :key="domain"
                class="domain-item"
              >
                <span class="domain-name">{{ domain }}</span>
                <button 
                  @click="removeDomain(domain)"
                  class="remove-button"
                  title="Remove domain"
                >
                  ×
                </button>
              </div>
            </div>
          </div>

          <div v-else class="empty-state">
            <p>No domains configured. Add domains above to restrict access.</p>
          </div>
        </div>
      </div>
    </div>

    <div class="actions">
      <button @click="saveSettings" class="save-button" :disabled="!hasChanges">
        Save Changes
      </button>
      <button @click="resetSettings" class="cancel-button">
        Cancel
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Reactive data
const restrictionMode = ref('allow-all')
const allowedDomains = ref([])
const newDomain = ref('')
const isProviderEnabled = ref(true)
const originalSettings = ref({})

// Computed properties
const isValidDomain = computed(() => {
  const domain = newDomain.value.trim()
  const domainRegex = /^[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/
  return domain && domainRegex.test(domain) && !allowedDomains.value.includes(domain)
})

const hasChanges = computed(() => {
  return JSON.stringify({
    restrictionMode: restrictionMode.value,
    allowedDomains: allowedDomains.value
  }) !== JSON.stringify(originalSettings.value)
})

// Methods
const addDomain = () => {
  if (isValidDomain.value) {
    allowedDomains.value.push(newDomain.value.trim().toLowerCase())
    newDomain.value = ''
  }
}

const removeDomain = (domain) => {
  const index = allowedDomains.value.indexOf(domain)
  if (index > -1) {
    allowedDomains.value.splice(index, 1)
  }
}

const saveSettings = () => {
  // API call to save settings would go here
  console.log('Saving settings:', {
    restrictionMode: restrictionMode.value,
    allowedDomains: allowedDomains.value
  })
  
  originalSettings.value = {
    restrictionMode: restrictionMode.value,
    allowedDomains: [...allowedDomains.value]
  }
  
  // Show success message
  alert('Domain restriction settings saved successfully!')
}

const resetSettings = () => {
  restrictionMode.value = originalSettings.value.restrictionMode || 'allow-all'
  allowedDomains.value = [...(originalSettings.value.allowedDomains || [])]
  newDomain.value = ''
}

// Load initial settings
onMounted(() => {
  // This would typically load from an API
  const initialSettings = {
    restrictionMode: 'allow-all',
    allowedDomains: []
  }
  
  restrictionMode.value = initialSettings.restrictionMode
  allowedDomains.value = initialSettings.allowedDomains
  originalSettings.value = { ...initialSettings }
})
</script>

<style scoped>
.domain-restriction-settings {
  max-width: 800px;
  padding: 24px;
  background: #fff;
  border-radius: 8px;
  border: 1px solid #e1e5e9;
}

.settings-header {
  margin-bottom: 32px;
}

.settings-header h3 {
  margin: 0 0 8px 0;
  font-size: 20px;
  font-weight: 600;
  color: #1a1a1a;
}

.settings-description {
  margin: 0;
  color: #6b7280;
  font-size: 14px;
}

.auth-provider-section {
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 24px;
}

.provider-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.provider-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.provider-icon {
  font-size: 18px;
}

.provider-name {
  font-weight: 500;
  font-size: 16px;
}

.provider-status {
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 12px;
  font-weight: 500;
  background: #f3f4f6;
  color: #6b7280;
}

.provider-status.active {
  background: #d1fae5;
  color: #065f46;
}

.radio-group {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 24px;
}

.radio-option {
  display: flex;
  flex-direction: column;
  gap: 4px;
  cursor: pointer;
  padding: 16px;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  transition: all 0.2s;
}

.radio-option:hover {
  border-color: #3b82f6;
  background: #f8fafc;
}

.radio-option input[type="radio"] {
  margin-right: 8px;
}

.radio-label {
  font-weight: 500;
  color: #1a1a1a;
}

.radio-description {
  font-size: 14px;
  color: #6b7280;
  margin-left: 20px;
}

.domain-management {
  border-top: 1px solid #e1e5e9;
  padding-top: 20px;
}

.input-label {
  display: block;
  font-weight: 500;
  margin-bottom: 8px;
  color: #1a1a1a;
}

.domain-input-group {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
}

.domain-input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
}

.domain-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.add-button {
  padding: 8px 16px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.add-button:hover:not(:disabled) {
  background: #2563eb;
}

.add-button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.input-hint {
  font-size: 12px;
  color: #6b7280;
  margin: 0;
}

.domains-list {
  margin-top: 20px;
}

.list-header {
  font-size: 14px;
  font-weight: 500;
  color: #1a1a1a;
  margin-bottom: 12px;
}

.domain-items {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.domain-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #f8fafc;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
}

.domain-name {
  font-family: monospace;
  font-size: 14px;
}

.remove-button {
  background: none;
  border: none;
  color: #ef4444;
  font-size: 18px;
  cursor: pointer;
  padding: 0;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.remove-button:hover {
  background: #fee2e2;
}

.empty-state {
  text-align: center;
  padding: 20px;
  color: #6b7280;
  font-style: italic;
}

.actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding-top: 20px;
  border-top: 1px solid #e1e5e9;
}

.save-button {
  padding: 10px 20px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.save-button:hover:not(:disabled) {
  background: #2563eb;
}

.save-button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.cancel-button {
  padding: 10px 20px;
  background: white;
  color: #6b7280;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.cancel-button:hover {
  background: #f9fafb;
  border-color: #9ca3af;
}
</style>