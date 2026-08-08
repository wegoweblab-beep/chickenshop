# chickenshop
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>美味雞肉線上訂購</title>
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen pb-28">
  <div id="app" class="max-w-md mx-auto bg-white min-h-screen shadow-md relative">
    
    <!-- 頁首 -->
    <header class="bg-amber-600 text-white p-4 text-center font-bold text-xl">
      🍗 鮮嫩雞肉專賣店 線上預訂
    </header>

    <div class="p-4 space-y-6">
      <!-- 1. 選擇門市與時間 -->
      <section class="bg-amber-50 p-4 rounded-xl border border-amber-200 space-y-3">
        <h2 class="font-bold text-amber-900 border-b border-amber-200 pb-1">1. 選擇取貨門市與時間</h2>
        
        <div>
          <label class="block text-sm font-medium mb-1">選擇門市</label>
          <select v-model="order.store" class="w-full p-2 border rounded-lg bg-white">
            <option value="旗艦店">旗艦店（市區）</option>
            <option value="二號店">二號店（高鐵站前）</option>
          </select>
        </div>

        <div class="grid grid-cols-2 gap-2">
          <div>
            <label class="block text-sm font-medium mb-1">取貨日期</label>
            <input type="date" v-model="order.date" class="w-full p-2 border rounded-lg bg-white">
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">取貨時段</label>
            <select v-model="order.timeSlot" class="w-full p-2 border rounded-lg bg-white">
              <option v-for="slot in availableSlots" :key="slot.time" :value="slot.time" :disabled="!slot.active || slot.full">
                {{ slot.time }} {{ !slot.active ? '(已關閉)' : slot.full ? '(已額滿)' : '' }}
              </option>
            </select>
          </div>
        </div>
      </section>

      <!-- 2. 選擇商品與處理方式 -->
      <section class="space-y-4">
        <h2 class="font-bold text-gray-800 text-lg">2. 選擇商品與加工方式</h2>
        
        <div v-for="product in menu" :key="product.id" class="border rounded-xl p-4 space-y-3 bg-white shadow-sm">
          <div class="flex justify-between items-center">
            <div>
              <div class="font-bold text-base">{{ product.name }}</div>
              <div class="text-amber-600 font-bold">${{ product.price }}</div>
            </div>
          </div>

          <!-- 切／不切 選擇選項 -->
          <div class="flex items-center justify-between pt-2 border-t">
            <span class="text-sm font-medium text-gray-700">加工選擇：</span>
            <div class="flex gap-4">
              <label class="inline-flex items-center cursor-pointer">
                <input type="radio" :name="'cut-' + product.id" value="要切" v-model="product.tempCut" class="text-amber-600 focus:ring-amber-500">
                <span class="ml-1 text-sm font-bold text-amber-800">要切</span>
              </label>
              <label class="inline-flex items-center cursor-pointer">
                <input type="radio" :name="'cut-' + product.id" value="不切" v-model="product.tempCut" class="text-amber-600 focus:ring-amber-500">
                <span class="ml-1 text-sm font-bold text-gray-600">不切</span>
              </label>
            </div>
          </div>

          <button @click="addToCart(product)" class="w-full bg-amber-500 hover:bg-amber-600 text-white font-bold py-2 rounded-lg text-sm transition">
            + 加入購物車 ({{ product.tempCut }})
          </button>
        </div>
      </section>

      <!-- 3. 購物車明細 -->
      <section v-if="cart.length > 0" class="border-t pt-4 space-y-4">
        <h2 class="font-bold text-gray-800 text-lg">3. 購物車清單</h2>
        
        <div v-for="(item, index) in cart" :key="index" class="bg-gray-50 p-3 rounded-lg border space-y-2">
          <div class="flex justify-between items-center">
            <div>
              <span class="font-bold text-gray-800">{{ item.name }}</span>
              <!-- 顯示要切/不切標籤 -->
              <span class="ml-2 px-2 py-0.5 text-xs font-bold rounded-full" :class="item.cutOption === '要切' ? 'bg-amber-100 text-amber-800' : 'bg-gray-200 text-gray-700'">
                {{ item.cutOption }}
              </span>
            </div>
            <button @click="removeFromCart(index)" class="text-red-500 text-xs hover:underline">刪除</button>
          </div>
          
          <div class="flex justify-between items-center text-sm">
            <span class="text-amber-600 font-bold">${{ item.price * item.qty }}</span>

            <!-- 數量調整與切法動態修改 -->
            <div class="flex items-center gap-3">
              <div class="flex items-center gap-1 bg-white border rounded">
                <button @click="item.qty > 1 ? item.qty-- : null" class="w-6 h-6 text-gray-600 font-bold">-</button>
                <span class="px-2">{{ item.qty }}</span>
                <button @click="item.qty++" class="w-6 h-6 text-gray-600 font-bold">+</button>
              </div>
            </div>
          </div>
        </div>

        <!-- 備註與聯絡資訊 -->
        <div class="space-y-3 pt-2">
          <div>
            <label class="block text-sm font-medium mb-1">訂單備註</label>
            <textarea v-model="order.note" placeholder="例如：醬汁分開放、拜拜用" class="w-full p-2 border rounded-lg focus:ring-1 focus:ring-amber-500"></textarea>
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">訂購人姓名 <span class="text-red-500">*</span></label>
            <input type="text" v-model="order.customerName" placeholder="請輸入姓名" class="w-full p-2 border rounded-lg">
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">聯絡電話 <span class="text-red-500">*</span></label>
            <input type="tel" v-model="order.customerPhone" placeholder="09xx-xxx-xxx" class="w-full p-2 border rounded-lg">
          </div>
        </div>
      </section>
    </div>

    <!-- 底部固定結帳列 -->
    <div v-if="cart.length > 0" class="fixed bottom-0 max-w-md w-full bg-white border-t p-4 flex justify-between items-center shadow-lg z-10">
      <div>
        <div class="text-xs text-gray-500">總金額</div>
        <div class="text-xl font-bold text-amber-600">${{ totalPrice }}</div>
      </div>
      <button @click="submitOrder" class="bg-amber-600 text-white font-bold px-6 py-3 rounded-xl hover:bg-amber-700 transition">
        送出預訂
      </button>
    </div>
  </div>

  <script>
    const { createApp, ref, computed } = Vue;
    createApp({
      setup() {
        const order = ref({
          store: '旗艦店',
          date: new Date().toISOString().split('T')[0],
          timeSlot: '11:30 - 12:00',
          customerName: '',
          customerPhone: '',
          note: ''
        });

        // 預設每個商品都有 tempCut (預設要切)
        const menu = ref([
          { id: 1, name: '招牌鹽水雞（全雞）', price: 450, tempCut: '要切' },
          { id: 2, name: '海南雞（半雞）', price: 250, tempCut: '要切' },
          { id: 3, name: '蔗香燻雞（半雞）', price: 260, tempCut: '要切' }
        ]);

        const availableSlots = ref([
          { time: '11:00 - 11:30', active: true, full: false },
          { time: '11:30 - 12:00', active: true, full: false },
          { time: '12:00 - 12:30', active: true, full: true },
          { time: '17:00 - 17:30', active: false, full: false }
        ]);

        const cart = ref([]);

        // 加入購物車（判斷商品名稱與「要切/不切」屬性）
        const addToCart = (product) => {
          const existingItem = cart.value.find(
            item => item.id === product.id && item.cutOption === product.tempCut
          );

          if (existingItem) {
            existingItem.qty++;
          } else {
            cart.value.push({
              id: product.id,
              name: product.name,
              price: product.price,
              qty: 1,
              cutOption: product.tempCut
            });
          }
        };

        const removeFromCart = (index) => {
          cart.value.splice(index, 1);
        };

        const totalPrice = computed(() => {
          return cart.value.reduce((sum, item) => sum + (item.price * item.qty), 0);
        });

        const submitOrder = () => {
          if (!order.value.customerName || !order.value.customerPhone) {
            alert('請填寫姓名與聯絡電話！');
            return;
          }
          alert(`訂單送出成功！\n總金額：$${totalPrice.value}\n取貨門市：${order.value.store}\n取貨時間：${order.value.date} ${order.value.timeSlot}`);
        };

        return { order, menu, availableSlots, cart, addToCart, removeFromCart, totalPrice, submitOrder };
      }
    }).mount('#app');
  </script>
</body>
</html>
