<template>
  <div class="subnet-calculator">
    <h2>Калькулятор подсетей</h2>
    
    <form @submit.prevent="calculate" class="calculator-form">
      <div class="form-group">
        <label for="ip-address">IP адрес:</label>
        <input
          id="ip-address"
          v-model="ipAddress"
          type="text"
          placeholder="192.168.1.150"
          :class="{ 'error': !isValidIp && ipAddress }"
          @keyup.enter="calculate"
        />
        <span v-if="!isValidIp && ipAddress" class="error-message">
          Неверный формат IP адреса
        </span>
      </div>

      <div class="form-group">
        <label for="netmask">Маска подсети:</label>
        <select
          id="netmask"
          v-model="selectedMask"
          @keyup.enter="calculate"
        >
          <option v-for="option in MASK_OPTIONS" :key="option.value" :value="option.value">
            {{ option.label }}
          </option>
        </select>
      </div>

      <button
        type="submit"
        :disabled="!isFormValid"
        class="calculate-btn"
      >
        Рассчитать
      </button>
    </form>

    <div v-if="results" class="results">
      <h3>Результаты:</h3>
      <div class="result-item">
        <strong>IP адрес:</strong> {{ results.ip }}
      </div>
      <div class="result-item">
        <strong>Маска:</strong> {{ results.mask }}
      </div>
      <div class="result-item">
        <strong>Адрес сети:</strong> {{ results.networkAddress }}
      </div>
      <div class="result-item">
        <strong>Количество адресов:</strong> {{ results.addressesCount }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { MASK_OPTIONS } from '../../utils/maskOptions'
import { isIpValid } from '../../utils/ipValidation'
import { getNetworkAddress, getAddressesCount } from '../../utils/networkCalculations'

interface CalculationResult {
  ip: string
  mask: string
  networkAddress: string
  addressesCount: number
}

const ipAddress = ref('192.168.1.150')
const selectedMask = ref('255.255.255.0')
const results = ref<CalculationResult | null>(null)

const isValidIp = computed(() => isIpValid(ipAddress.value))
const isFormValid = computed(() => isValidIp.value && selectedMask.value)

function calculate() {
  if (!isFormValid.value) return

  results.value = {
    ip: ipAddress.value,
    mask: selectedMask.value,
    networkAddress: getNetworkAddress(ipAddress.value, selectedMask.value),
    addressesCount: getAddressesCount(selectedMask.value)
  }
}
</script>

<style scoped>
.subnet-calculator {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.calculator-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 24px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

label {
  font-weight: bold;
  color: var(--color-text);
}

input, select {
  padding: 8px 12px;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  font-size: 14px;
}

input.error {
  border-color: var(--color-error);
}

.error-message {
  color: var(--color-error);
  font-size: 12px;
}

.calculate-btn {
  padding: 12px 24px;
  background-color: var(--color-primary);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.calculate-btn:hover:not(:disabled) {
  background-color: var(--color-primary-dark);
}

.calculate-btn:disabled {
  background-color: var(--color-gray);
  cursor: not-allowed;
}

.results {
  padding: 16px;
  background-color: var(--color-background-soft);
  border-radius: 8px;
  border: 1px solid var(--color-border);
}

.results h3 {
  margin-top: 0;
  margin-bottom: 16px;
  color: var(--color-heading);
}

.result-item {
  margin-bottom: 8px;
  padding: 4px 0;
}
</style>