<script setup>
import QrcodeVue from 'qrcode.vue'
import { ref } from 'vue'

// State
const url = ref('https://buildit.com')
const qrCode = ref('https://buildit.com')
const color = ref('#000000')
const backgroundColor = ref('#ffffff')
const size = ref(200)
const errorCorrection = ref('M')

// Methods
function generateQRCode() {
  qrCode.value = url.value
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-white p-4 dark:bg-[#141c26]">
    <div class="max-w-2xl w-full overflow-hidden rounded-lg bg-white shadow-md dark:bg-[#141c26]">
      <div class="border-b p-6">
        <h1 class="text-center text-2xl font-bold">
          QR Code Generator
        </h1>
      </div>
      <div class="p-6">
        <form class="space-y-4" @submit.prevent="generateQRCode">
          <div>
            <label for="url" class="mb-1 block text-sm font-medium">URL</label>
            <input
              id="url"
              v-model="url"
              type="url"
              placeholder="Enter a URL"
              required
              class="w-full border bg-white
dark:bg-[#141c26] rounded-md px-3 py-2 focus:outline-primary focus:ring-2 focus:ring-primary"
            >
          </div>
          <div>
            <label for="backgroundColor" class="mb-1 block text-sm font-medium">Background Color</label>
            <input
              id="backgroundColor"
              v-model="backgroundColor"
              type="color"
              class="h-10 w-full border bg-white
dark:bg-[#141c26] focus:outline-primary focus:outline-2 rounded-md p-0"
            >
          </div>
          <div>
            <label for="size" class="mb-1 block text-sm font-medium">
              Size: {{ size }}x{{ size }}
            </label>
            <input
              id="size"
              v-model.number="size"
              type="range"
              min="100"
              max="400"
              step="10"
              class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-gray-200"
            >
          </div>
          <div>
            <label for="errorCorrection" class="mb-1 block text-sm font-medium">Error Correction Level</label>
            <select
              id="errorCorrection"
              v-model="errorCorrection"
              class="w-full border rounded-md px-3 py-2 focus:outline-none bg-white
dark:bg-[#141c26] focus:ring-2 focus:ring-primary"
            >
              <option value="L">
                Low (7%)
              </option>
              <option value="M">
                Medium (15%)
              </option>
              <option value="Q">
                Quartile (25%)
              </option>
              <option value="H">
                High (30%)
              </option>
            </select>
          </div>
          <button
            type="submit"
            class="w-full rounded-md bg-primary px-4 py-2 text-black transition-colors hover:bg-primary/90"
          >
            Generate QR Code
          </button>
        </form>
      </div>
      <div v-if="qrCode" class="flex justify-center p-6">
        <div class="mt-4">
          <QrcodeVue
            :value="qrCode"
            :size="size"
            :color="color"
            :background="backgroundColor"
            :level="errorCorrection"
            :margin="10"
          />
        </div>
      </div>
    </div>
  </div>
</template>

  <style>
  :root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --border: 214.3 31.8% 91.4%;
  --input: 214.3 31.8% 91.4%;
  --ring: 215 20.2% 65.1%;
  --radius: 0.5rem;
}

.bg-primary {
  background-color: hsl(var(--primary));
}

.bg-primary\/90 {
  background-color: hsl(var(--primary) / 0.9);
}

.text-primary {
  color: hsl(var(--primary));
}

.text-white {
  color: white;
}

.focus\:ring-primary:focus {
  --tw-ring-color: hsl(var(--primary));
}

.hover\:bg-primary\/90:hover {
  background-color: hsl(var(--primary) / 0.9);
}
</style>
