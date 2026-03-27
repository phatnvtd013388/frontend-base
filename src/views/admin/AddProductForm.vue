<template>
  <div class="admin-container">
    <div class="container py-5">
      <div class="row justify-content-center">
        <div class="col-12 col-md-8 col-lg-6">
          <div class="form-card">
            <h2 class="form-title text-center mb-4">Thêm Đồ Uống Mới</h2>
            
            <!-- Hiển thị thông báo (Alerts) -->
            <div v-if="alertMessage" :class="`alert alert-${alertType} alert-dismissible fade show`" role="alert">
              {{ alertMessage }}
              <button type="button" class="btn-close" @click="alertMessage = ''" aria-label="Close"></button>
            </div>
            
            <form @submit.prevent="submitForm" novalidate>
              <!-- Tên đồ uống -->
              <div class="mb-3">
                <label for="productName" class="form-label fw-bold">Tên đồ uống <span class="text-danger">*</span></label>
                <input 
                  type="text" 
                  class="form-control" 
                  id="productName" 
                  v-model="formData.name"
                  :class="{ 'is-invalid': errors.name }"
                  placeholder="Ví dụ: Cà phê sữa đá"
                />
                <div class="invalid-feedback" v-if="errors.name">{{ errors.name }}</div>
              </div>

              <!-- Giá bán -->
              <div class="mb-3">
                <label for="productPrice" class="form-label fw-bold">Giá bán (VNĐ) <span class="text-danger">*</span></label>
                <input 
                  type="number" 
                  class="form-control" 
                  id="productPrice" 
                  v-model="formData.price"
                  :class="{ 'is-invalid': errors.price }"
                  placeholder="Ví dụ: 35000"
                />
                <div class="invalid-feedback" v-if="errors.price">{{ errors.price }}</div>
              </div>

              <!-- Danh mục (Dropdown) -->
              <div class="mb-3">
                <label for="productCategory" class="form-label fw-bold">Danh mục <span class="text-danger">*</span></label>
                <select 
                  class="form-select" 
                  id="productCategory" 
                  v-model="formData.category"
                  :class="{ 'is-invalid': errors.category }"
                >
                  <option value="" disabled selected>-- Chọn danh mục --</option>
                  <option value="Iced coffee">Iced coffee</option>
                  <option value="Milk-Based Coffee">Milk-Based Coffee</option>
                  <option value="Classic Coffee">Classic Coffee</option>
                  <option value="Specialty Coffee">Specialty Coffee</option>
                  <option value="Sweet Coffee">Sweet Coffee</option>
                </select>
                <div class="invalid-feedback" v-if="errors.category">{{ errors.category }}</div>
              </div>

              <!-- Link Ảnh -->
              <div class="mb-3">
                <label for="productImage" class="form-label fw-bold">Link Ảnh URL <span class="text-danger">*</span></label>
                <input 
                  type="url" 
                  class="form-control" 
                  id="productImage" 
                  v-model="formData.image"
                  :class="{ 'is-invalid': errors.image }"
                  placeholder="https://images.unsplash.com/..."
                />
                <div class="invalid-feedback" v-if="errors.image">{{ errors.image }}</div>
                
                <!-- Hiển thị ảnh thu nhỏ (Preview) nếu có link hợp lệ -->
                <div v-if="formData.image && !errors.image" class="mt-2 text-center">
                  <img :src="formData.image" alt="Preview ảnh" class="img-preview rounded shadow-sm" @error="handleImageError" />
                </div>
              </div>

              <!-- Mô tả -->
              <div class="mb-4">
                <label for="productDesc" class="form-label fw-bold">Mô tả chi tiết <span class="text-danger">*</span></label>
                <textarea 
                  class="form-control" 
                  id="productDesc" 
                  rows="3" 
                  v-model="formData.description"
                  :class="{ 'is-invalid': errors.description }"
                  placeholder="Mô tả hương vị, thành phần, cách pha chế..."
                ></textarea>
                <div class="invalid-feedback" v-if="errors.description">{{ errors.description }}</div>
              </div>

              <!-- Nút Action -->
              <div class="d-flex gap-3 justify-content-end mt-4">
                <button type="button" class="btn btn-outline-secondary px-4 fw-medium btn-cancel" @click="handleCancel">
                  Hủy
                </button>
                <button type="submit" class="btn btn-primary px-4 fw-medium btn-save" :disabled="isSubmitting">
                  <span v-if="isSubmitting" class="spinner-border spinner-border-sm me-2" role="status" aria-hidden="true"></span>
                  Lưu / Thêm mới
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

const router = useRouter()

// Trạng thái form
const formData = reactive({
  name: '',
  price: '',
  category: '',
  image: '',
  description: ''
})

// Trạng thái bắt lỗi front-end
const errors = reactive({
  name: '',
  price: '',
  category: '',
  image: '',
  description: ''
})

// Trạng thái loading và thông báo
const isSubmitting = ref(false)
const alertMessage = ref('')
const alertType = ref('success') // 'success' màu xanh, 'danger' màu đỏ

// Validation Form Front-end
const validateForm = () => {
  let isValid = true
  
  // Reset lỗi mỗi lần bấm Submit
  Object.keys(errors).forEach(key => errors[key] = '')

  // 1. Tên bắt buộc
  if (!formData.name.trim()) {
    errors.name = 'Tên đồ uống không được để trống.'
    isValid = false
  }

  // 2. Giá bắt buộc & lớn hơn 0 (Chuẩn bị RQ13)
  if (!formData.price) {
    errors.price = 'Vui lòng nhập giá bán.'
    isValid = false
  } else if (Number(formData.price) <= 0) {
    errors.price = 'Giá bán bắt buộc phải lớn hơn 0.' // Thỏa mãn RQ13
    isValid = false
  }

  // 3. Phải chọn Danh mục
  if (!formData.category) {
    errors.category = 'Vui lòng chọn một danh mục.'
    isValid = false
  }

  // 4. Bắt buộc Link ảnh
  if (!formData.image.trim()) {
    errors.image = 'Link ảnh không được để trống.'
    isValid = false
  }

  // 5. Bắt buộc Mô tả
  if (!formData.description.trim()) {
    errors.description = 'Mô tả không được để trống.'
    isValid = false
  }

  return isValid
}

// Xử lý gửi Form (Submit)
const submitForm = async () => {
  // Chạy hàm validate trước, nếu có lỗi thì không cho gửi API
  if (!validateForm()) return

  isSubmitting.value = true
  alertMessage.value = ''

  try {
    // Ép kiểu Json (Price thành số) theo quy chuẩn trước khi đẩy lên Backend
    const payload = {
      ...formData,
      price: Number(formData.price)
    }

    // Gọi phương thức POST bằng Axios vào API endpoint thật (hoặc giả lập)
    const response = await axios.post('https://jsonplaceholder.typicode.com/posts', payload)

    // Xử lý Thành công (Khi HTTP Status là 200/201 - JSONPlaceholder trả về 201 cho POST)
    if (response.status === 200 || response.status === 201) {
      alertType.value = 'success'
      alertMessage.value = 'Thêm đồ uống thành công!'
      
      // Xoá trắng lại Form sau khi thêm
      resetForm()
      
      // Hoặc chuyển hướng (Chờ 2 giây rồi chuyển hướng)
      // setTimeout(() => router.push('/'), 2000)
    }
  } catch (error) {
    // Xử lý Thất bại (Status 400/500) và Bắt lỗi catch
    alertType.value = 'danger'
    
    // Giả lập thông báo đỏ (Trùng món/Sever lỗi theo checklist)
    if (error.response && error.response.status === 400) {
      alertMessage.value = 'Thêm thất bại: Trùng tên món!' // Có thể tùy biến dựa vào error message từ backend
    } else {
      alertMessage.value = 'Thêm thất bại: Lỗi server!'
    }
    console.error("Lỗi khi thêm đồ uống:", error)
  } finally {
    isSubmitting.value = false
  }
}

// Hàm huỷ bỏ
const handleCancel = () => {
  router.back() // Quay lại trang cũ
}

// Hàm Xóa trắng các hộp nhập (Reset)
const resetForm = () => {
  formData.name = ''
  formData.price = ''
  formData.category = ''
  formData.image = ''
  formData.description = ''
  Object.keys(errors).forEach(key => errors[key] = '')
}

// Hàm bắt lỗi khi ảnh URL bị hư (hiện ảnh vỡ / fallback ảnh)
const handleImageError = () => {
  errors.image = 'Đường dẫn ảnh bị lỗi, không thể hiển thị ảnh này.'
}

</script>

<style scoped>
.admin-container {
  min-height: 100vh;
  background-color: #f5ede0; /* Màu nền chủ đạo quán cà phê */
}

.form-card {
  background-color: #ffffff;
  border-radius: 20px;
  padding: 2.5rem 3rem;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.05);
  border: 1px solid #ede0ce;
}

.form-title {
  color: #2c1a0e;
  font-weight: 800;
  font-size: 28px;
  margin-bottom: 2rem;
}

.form-label {
  color: #4a3a2e;
  font-size: 15px;
}

.form-control, .form-select {
  border-radius: 12px;
  border: 1px solid #ddd;
  padding: 12px 16px;
  background-color: #fafafa;
  transition: all 0.3s ease;
}

.form-control:focus, .form-select:focus {
  border-color: #2c6fad;
  box-shadow: 0 0 0 0.25rem rgba(44, 111, 173, 0.25);
  background-color: #fff;
}

/* Hiệu ứng báo lỗi (Màu đỏ) */
.is-invalid {
  border-color: #dc3545 !important;
  background-color: #fff8f8 !important;
}
.invalid-feedback {
  font-size: 13px;
  font-weight: 500;
  margin-top: 6px;
}

.img-preview {
  max-width: 100%;
  max-height: 200px;
  object-fit: contain;
  border: 2px dashed #ede0ce;
  padding: 4px;
}

.btn-primary.btn-save {
  background-color: #2c6fad;
  border-color: #2c6fad;
  border-radius: 30px;
  padding: 10px 24px;
}

.btn-primary.btn-save:hover, .btn-primary.btn-save:focus {
  background-color: #1b5490;
  border-color: #1b5490;
  box-shadow: 0 6px 15px rgba(44, 111, 173, 0.3);
  transform: translateY(-2px);
}

.btn-secondary.btn-cancel, .btn-outline-secondary.btn-cancel {
  border-radius: 30px;
  padding: 10px 24px;
}

/* Responsiveness */
@media (max-width: 768px) {
  .form-card {
    padding: 1.5rem 1.5rem;
  }
}
</style>
