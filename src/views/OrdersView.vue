<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import { getAllOrders, createNewOrder, getAllCars } from '../api/requests'

interface Car {
  id: number
  name: string
  price: number
}

interface Order {
  name: string
  phone_number: string
  email: string
  car: Car
}

const orders = ref<Order[]>([])
const isLoading = ref(false)
const showCreateForm = ref(false)
const hasNextPage = ref(true)
const isCreating = ref(false)
const limit = 3
const offset = ref(0)

const name = ref('')
const phone_number = ref('')
const email = ref('')
const car_id = ref<number | null>(null)

const cars = ref<Car[]>([])

const buildQuery = () => {
  const params = new URLSearchParams()
  params.append('limit', limit.toString())
  params.append('offset', offset.value.toString())
  return params.toString()
}

const fetchCars = async () => {
  try {
    const response = await getAllCars('')
    cars.value = response.results
  } catch (error) {
    console.error(error)
  }
}

const fetchOrders = async (append = false) => {
  try {
    isLoading.value = true
    const query = buildQuery()
    const response = await getAllOrders(`?${query}`)

    if (append) {
      orders.value.push(...response.results)
    } else {
      orders.value = response.results
    }

    hasNextPage.value = response.next_page
  } catch (error) {
    console.error(error)
  } finally {
    isLoading.value = false
  }
}

const createOrder = async () => {
  if (!name.value.trim() || !phone_number.value.trim() || !email.value.trim() || !car_id.value) return

  try {
    isCreating.value = true
    const payload = {
      name: name.value.trim(),
      phone_number: phone_number.value.trim(),
      email: email.value.trim(),
      car_id: car_id.value
    }
    await createNewOrder(payload)

    name.value = ''
    phone_number.value = ''
    email.value = ''
    car_id.value = null
    showCreateForm.value = false

    offset.value = 0
    await fetchOrders()
  } catch (error) {
    console.error(error)
  } finally {
    isCreating.value = false
  }
}

const loadNextPage = () => {
  if (!hasNextPage.value || isLoading.value) return
  offset.value += limit
  fetchOrders(true)
}

onMounted(() => {
  fetchCars()
  fetchOrders()
})

watch(showCreateForm, (value) => {
  document.body.style.overflow = value ? 'hidden' : ''
})
</script>

<template>
        <img
    alt="background"
    class="background"
    src="../assets/backgrounds/background-catalog.jpg"
  />
  <div class="order-container">
    <!-- <div class="create-section">
      <button class="create-btn" @click="showCreateForm = true">
        Добавить заказ
      </button>
    </div> -->

    <Teleport to="body">
      <div v-if="showCreateForm" class="modal-overlay" @click.self="showCreateForm = false">
        <div class="modal">
          <button class="modal-close" @click="showCreateForm = false">✕</button>
          <h2 class="modal-title">Добавить заказ</h2>

          <input v-model="name" type="text" placeholder="Имя" class="input modal-input" />
          <input v-model="phone_number" type="text" placeholder="Телефон" class="input modal-input" />
          <input v-model="email" type="email" placeholder="Email" class="input modal-input" />

          <select v-model="car_id" class="input modal-input">
            <option disabled value="">Выберите авто</option>
            <option
              v-for="(car, index) in cars"
              :key="car.id"
              :value="index + 1"
            >
              {{ car.name }} - {{ car.price }} ₽
            </option>
          </select>

          <button class="confirm-btn modal-confirm" :disabled="isCreating" @click="createOrder">
            {{ isCreating ? 'Создание...' : 'Подтвердить' }}
          </button>
        </div>
      </div>
    </Teleport>

    <div v-if="isLoading" class="loading">Загрузка...</div>
    <div v-else-if="!orders.length" class="empty">Заказов нет</div>

    <div v-else class="orders">
      <div v-for="(order, index) in orders" :key="index" class="order-card">
        <div class="order-header">
          <h2>{{ order.name }}</h2>
          <span class="price">{{ order.car.price }} ₽</span>
        </div>
        <div class="order-info">
          <p><strong>Email:</strong> {{ order.email }}</p>
          <p><strong>Телефон:</strong> {{ order.phone_number }}</p>
          <p><strong>Авто:</strong> {{ order.car.name }}</p>
        </div>
      </div>
    </div>

    <div class="pagination" v-if="hasNextPage">
      <button class="load-more" :disabled="isLoading" @click="loadNextPage">
        Загрузить ещё
      </button>
    </div>
  </div>
</template>

<style scoped>
.background {
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  object-fit: cover;
  z-index: -1;
}

.order-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 24px;
  font-family: 'Inter', sans-serif;
}

.create-section {
  display: flex;
  justify-content: center;
  margin-bottom: 32px;
}

.create-btn {
  padding: 14px 32px;
  border-radius: 999px;
  border: 1px solid #000;
  background: linear-gradient(90deg, #000, #333);
  color: #fff;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.25s ease;
}

.create-btn:hover {
  background: linear-gradient(90deg, #f9a825, #ffc107);
  color: #000;
}

.orders {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.order-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.order-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 36px rgba(0, 0, 0, 0.15);
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.order-header h2 {
  font-size: 20px;
  font-weight: 600;
  color: #333;
}

.price {
  font-weight: 700;
  color: #f9a825;
  font-size: 18px;
}

.order-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.order-info p {
  margin: 0;
  font-size: 14px;
  color: #555;
}

.load-more {
  padding: 12px 28px;
  border-radius: 999px;
  border: none;
  background: #000;
  color: #fff;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.25s ease;
}

.load-more:hover:not(:disabled) {
  background: #f9a825;
  color: #000;
}

.load-more:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.loading, .empty {
  text-align: center;
  font-size: 18px;
  color: #777;
}

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(6px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  position: relative;
  width: 420px;
  padding: 32px;
  background: #fff;
  border-radius: 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.25);
  animation: modalIn 0.25s ease;
}

@keyframes modalIn {
  from {
    transform: translateY(20px) scale(0.98);
    opacity: 0;
  }
  to {
    transform: none;
    opacity: 1;
  }
}

.modal-close {
  position: absolute;
  top: 14px;
  right: 14px;
  border: none;
  background: transparent;
  font-size: 18px;
  cursor: pointer;
}

.modal-title {
  margin-bottom: 8px;
  font-size: 20px;
  font-weight: 500;
  text-align: center;
}

.modal-input {
  width: 100%;
  padding: 12px 16px;
  border-radius: 12px;
  border: 1px solid #ddd;
  background: #fff;
  color: #111;
  font-size: 14px;
  outline: none;
  transition: all 0.25s ease;
}

.modal-confirm {
  margin-top: 12px;
  width: 100%;
  padding: 12px 28px;
  border-radius: 999px;
  border: 1px solid #000;
  background: transparent;
  color: #000;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.25s ease;
}

.modal-confirm:hover:not(:disabled) {
  background: #000;
  color: #fff;
}

.modal-confirm:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>