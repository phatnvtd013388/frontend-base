<template>
  <RouterLink :to="'/product/' + product.ID" class="product-card" v-if="product">
    <!-- Hình ảnh trong nền trắng -->
    <div class="image-box">
      <img :src="product.Image" :alt="product.Name" class="product-image" />
    </div>

    <!-- Thông tin sản phẩm -->
    <div class="product-info">
      <h5 class="product-name">{{ product.Name }}</h5>
      
      <p class="product-desc" v-if="product.Description">
        {{ product.Description }}
      </p>
      <p class="product-desc" v-else>
        Cà phê nguyên chất rang xay mỗi ngày, đậm đà hương vị truyền thống.
      </p>

      <div class="d-flex flex-wrap justify-content-between align-items-center mt-auto pt-3 gap-2">
        <span class="product-price">{{ formattedPrice }}</span>
        <button class="btn btn-primary btn-sm rounded-pill px-3 fw-medium action-btn text-nowrap">Add to Cart</button>
      </div>
    </div>
  </RouterLink>
  
  <div v-else class="product-card-skeleton">
    <p>Đang tải dữ liệu...</p>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  product: {
    type: Object,
    required: true
  }
})

// Định dạng giá tiền (VD: 35000 -> "35.000 ₫")
// Giữ nguyên định dạng tiền Việt vì đây là website bán hàng Việt Nam
const formattedPrice = computed(() => {
  if (!props.product || props.product.Price === undefined) return '0 ₫'
  return new Intl.NumberFormat('vi-VN', { 
    style: 'currency', 
    currency: 'VND' 
  }).format(props.product.Price)
})
</script>

<style scoped>
/* Màu nền kem/beige giống với hình tham khảo (Hình 2) */
.product-card {
  background-color: #eedab3;
  border-radius: 12px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  height: 100%;
  text-decoration: none;
  color: inherit;
}

.image-box {
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 1rem;
  width: 100%;
  aspect-ratio: 1 / 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.product-info {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.product-name {
  font-weight: 700;
  font-size: 1.1rem;
  color: #000;
  margin-bottom: 0.5rem;
}

.product-desc {
  font-size: 0.85rem;
  color: #333;
  line-height: 1.4;
  margin-bottom: 0;
  flex-grow: 1; /* Đẩy phần giá và nút bấm xuống dưới cùng */
}

.product-price {
  font-weight: 800;
  font-size: 1.15rem;
  color: #000;
}

/* Nút bấm xanh dương (mặc định của btn-primary) giống Image 2 */
.action-btn {
  font-size: 0.85rem;
  background-color: #0d6efd;
  border: none;
}
.action-btn:hover {
  background-color: #0b5ed7;
}

.product-card-skeleton {
  border: 1px solid #eee;
  border-radius: 12px;
  padding: 16px;
  text-align: center;
  color: #888;
  background-color: #fafafa;
}
</style>