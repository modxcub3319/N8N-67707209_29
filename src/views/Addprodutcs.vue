<script setup>
import { ref } from 'vue'

// 1. เตรียมข้อมูลสำหรับส่งไปที่ API
const product = ref({
  id: '',
  name: '',
  unit: 0,
  price: 0
})

const isLoading = ref(false)

// 2. ฟังก์ชันส่งข้อมูล (POST)
const submitProduct = async () => {
  // Validation เบื้องต้น
  if (!product.value.id || !product.value.name) {
    alert("กรุณากรอกรหัสและชื่อสินค้าให้ครบถ้วน")
    return
  }

  isLoading.value = true
  
  try {
    const res = await fetch("http://localhost:5678/webhook/add-product", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(product.value)
    })

    if (res.ok) {
      alert("บันทึกข้อมูลสำเร็จแล้ว!")
      // ล้างข้อมูลในฟอร์มหลังจากบันทึก
      product.value = { id: '', name: '', unit: 0, price: 0 }
    } else {
      throw new Error("เซิร์ฟเวอร์ตอบกลับด้วยข้อผิดพลาด")
    }
  } catch (error) {
    console.error("Error:", error)
    alert("ไม่สามารถบันทึกข้อมูลได้ กรุณาตรวจสอบการเชื่อมต่อ API")
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="container">
    <div class="card">
      <h2>📦 เพิ่มสินค้าใหม่</h2>
      <hr />
      
      <form @submit.prevent="submitProduct">
        <div class="form-group">
          <label>รหัสสินค้า</label>
          <input v-model="product.id" type="text" placeholder="ตัวอย่าง: P001" required />
        </div>

        <div class="form-group">
          <label>ชื่อสินค้า</label>
          <input v-model="product.name" type="text" placeholder="ระบุชื่อสินค้า" required />
        </div>

        <div class="row">
          <div class="form-group">
            <label>จำนวน</label>
            <input v-model.number="product.unit" type="number" min="0" />
          </div>

          <div class="form-group">
            <label>ราคา (บาท)</label>
            <input v-model.number="product.price" type="number" min="0" step="0.01" />
          </div>
        </div>

        <button type="submit" :disabled="isLoading" class="btn-submit">
          {{ isLoading ? 'กำลังบันทึก...' : 'บันทึกข้อมูล' }}
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  padding: 40px 20px;
  font-family: 'Inter', sans-serif;
}

.card {
  width: 100%;
  max-width: 500px;
  background: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

h2 {
  margin-bottom: 10px;
  color: #333;
}

hr {
  border: 0;
  border-top: 1px solid #eee;
  margin-bottom: 25px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  font-weight: 600;
  margin-bottom: 8px;
  color: #555;
}

input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  box-sizing: border-box; /* ป้องกัน input ล้นขอบ */
  font-size: 16px;
}

input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

.row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.btn-submit {
  width: 100%;
  padding: 14px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-submit:hover {
  background-color: #0056b3;
}

.btn-submit:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>