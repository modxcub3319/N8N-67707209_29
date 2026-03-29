<script setup>
import { ref, onMounted } from 'vue'

// 1. สร้างตัวแปร Reactive
const products = ref([])
const isLoading = ref(false)

// 2. ฟังก์ชันดึงข้อมูลจาก n8n
const fetchProducts = async () => {
  isLoading.value = true
  try {
    // หมายเหตุ: ตรวจสอบ URL ให้ตรงกับใน n8n (Webhook Node)
    // หาก Activate Workflow แล้วให้ใช้ /webhook/ ถ้ายังไม่ Activate ให้ใช้ /webhook-test/
    const res = await fetch("http://localhost:5678/webhook/data") 
    
    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`)
    }
    
    const data = await res.json()
    
    // ตรวจสอบว่าข้อมูลที่ได้เป็น Array หรือไม่
    products.value = Array.isArray(data) ? data : [data]
    
  } catch (error) {
    console.error('Failed to fetch products:', error)
    // alert('ไม่สามารถดึงข้อมูลได้: ' + error.message)
  } finally {
    // หน่วงเวลาเล็กน้อยเพื่อให้เห็นสถานะการโหลด (Optional)
    setTimeout(() => {
      isLoading.value = false
    }, 500)
  }
}

// 3. เรียกใช้งานเมื่อ Component ถูกติดตั้ง
onMounted(() => {
  fetchProducts()
})
</script>

<template>
  <div class="container">
    <div class="header-section">
      <h2>📦 รายการสินค้าในคลัง</h2>
      
      <button 
        @click="fetchProducts" 
        :disabled="isLoading"
        class="btn-reset"
      >
        <span v-if="isLoading" class="loader"></span>
        {{ isLoading ? 'กำลังโหลด...' : '🔄 รีเซ็ตข้อมูล' }}
      </button>
    </div>
    
    <div class="table-container">
      <table class="product-table">
        <thead>
          <tr>
            <th>รหัสสินค้า</th>
            <th>ชื่อสินค้า</th>
            <th>จำนวน (ชิ้น)</th>
            <th>ราคา (บาท)</th>
          </tr>
        </thead>
        
        <tbody>
          <tr v-for="item in products" :key="item.id">
            <td>{{ item.id || item.รหัสสินค้า }}</td>
            <td>{{ item.name || item.ชื่อสินค้า }}</td>
            <td>{{ item.unit || item.จำนวน }}</td>
            <td>{{ item.price || item.ราคา }}</td>
          </tr>

          <tr v-if="products.length === 0">
            <td colspan="4" class="empty-state">
              <div v-if="isLoading">กำลังดึงข้อมูลล่าสุดจาก Google Sheets...</div>
              <div v-else>ไม่พบข้อมูลสินค้าในคลัง</div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 40px 20px;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  max-width: 1000px;
  margin: 0 auto;
}

.header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

h2 {
  color: #2c3e50;
  margin: 0;
  font-size: 24px;
}

/* ปุ่ม Reset */
.btn-reset {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background-color: #f8f9fa;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  font-size: 14px;
  transition: all 0.2s ease;
}

.btn-reset:hover:not(:disabled) {
  background-color: #e9ecef;
  border-color: #ccc;
  transform: translateY(-1px);
}

.btn-reset:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

/* ตาราง */
.table-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

.product-table thead {
  background-color: #007bff;
  color: white;
}

.product-table th {
  padding: 15px;
  font-weight: 600;
  text-transform: uppercase;
  font-size: 13px;
  letter-spacing: 0.5px;
}

.product-table td {
  padding: 15px;
  border-bottom: 1px solid #eee;
  color: #444;
}

.product-table tr:last-child td {
  border-bottom: none;
}

.product-table tr:hover {
  background-color: #f8fbff;
}

.empty-state {
  text-align: center;
  padding: 40px !important;
  color: #888;
  font-style: italic;
}

/* Loading Spinner เล็กๆ ในปุ่ม */
.loader {
  width: 14px;
  height: 14px;
  border: 2px solid #333;
  border-bottom-color: transparent;
  border-radius: 50%;
  display: inline-block;
  animation: rotation 1s linear infinite;
}

@keyframes rotation {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>