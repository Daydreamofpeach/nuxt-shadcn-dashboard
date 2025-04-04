<script setup>
// Components would be imported from shadcn-vue
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'
import { useToast } from '@/components/ui/toast'

import { Clock, Image, MapPin } from 'lucide-vue-next'
import { computed, onMounted, ref, watch } from 'vue'
// etc.

const hourlyRates = {
  1: { price: 199, maxLocations: 2, defaultImages: 25 },
  2: { price: 349, maxLocations: 4, defaultImages: 60 },
  3: { price: 499, maxLocations: 7, defaultImages: 80 },
  4: { price: 549, maxLocations: 9, defaultImages: 110 },
  5: { price: 749, maxLocations: 10, defaultImages: 130 },
}

const ADDITIONAL_IMAGE_PRICE = 4.99
const MAX_QUOTES_PER_USER = 3

// State
const hours = ref(3)
const locations = ref(7)
const editedImages = ref(80)
const extraImages = ref(null)
const name = ref('')
const email = ref('')
const phone = ref('')
const showSummary = ref(false)
const quotesRequested = ref(0)
const quotes = ref([])
const showQuotaExhaustedDialog = ref(false)
const { toast } = useToast()

// Computed properties
const basePrice = computed(() => hourlyRates[hours.value].price)

const additionalImagesSlider = computed(() => {
  return Math.round((editedImages.value - hourlyRates[hours.value].defaultImages) / 5) * 5
})

const additionalImages = computed(() => {
  return extraImages.value !== null
    ? extraImages.value
    : Math.max(0, editedImages.value - hourlyRates[hours.value].defaultImages)
})

const additionalImagesCost = computed(() => {
  return additionalImages.value * ADDITIONAL_IMAGE_PRICE
})

const totalCost = computed(() => {
  return basePrice.value + additionalImagesCost.value
})

// Methods
function handleHoursChange(value) {
  hours.value = value
  editedImages.value = hourlyRates[value].defaultImages
  locations.value = Math.min(locations.value, hourlyRates[value].maxLocations)
  extraImages.value = null
}

function setLocations(value) {
  locations.value = Math.min(value, hourlyRates[hours.value].maxLocations)
}

function handleEditedImagesChange(value) {
  editedImages.value = hourlyRates[hours.value].defaultImages + value
  extraImages.value = null
}

function handleExtraImagesChange(value) {
  const numValue = Number.parseInt(value, 10)
  if (numValue >= 100 && numValue <= 500) {
    extraImages.value = numValue
    editedImages.value = hourlyRates[hours.value].defaultImages
  }
  else {
    extraImages.value = null
  }
}

function handleSubmit() {
  if (quotesRequested.value >= MAX_QUOTES_PER_USER) {
    showQuotaExhaustedDialog.value = true
    return
  }

  showSummary.value = true
  quotesRequested.value += 1

  if (email.value) {
    localStorage.setItem(`quotesRequested_${email.value}`, quotesRequested.value.toString())
  }

  quotes.value.push({
    name: name.value,
    email: email.value,
    phone: phone.value,
    hours: hours.value,
    locations: locations.value,
    editedImages: extraImages.value !== null
      ? extraImages.value + hourlyRates[hours.value].defaultImages
      : editedImages.value,
    totalCost: totalCost.value,
    date: new Date().toLocaleString(),
  })

  toast({
    title: 'Quote Generated',
    description: 'Your quote is now available in the summary section.',
  })
}

function resetForm() {
  hours.value = 1
  locations.value = 2
  editedImages.value = 25
  extraImages.value = null
  name.value = ''
  email.value = ''
  phone.value = ''
  showSummary.value = false
}

// Watch for changes that should reset the summary
watch([hours, locations, editedImages, extraImages], () => {
  if (showSummary.value) {
    showSummary.value = false
    name.value = ''
    email.value = ''
    phone.value = ''
  }
})

// Load saved quotes on mount
onMounted(() => {
  const storedQuotes = localStorage.getItem('quotes')
  if (storedQuotes) {
    quotes.value = JSON.parse(storedQuotes)
  }

  watch(() => email.value, (newEmail) => {
    if (newEmail) {
      const storedQuotesRequested = localStorage.getItem(`quotesRequested_${newEmail}`)
      if (storedQuotesRequested) {
        quotesRequested.value = Number.parseInt(storedQuotesRequested, 10)
      }
      else {
        quotesRequested.value = 0
      }
    }
  })

  // Save quotes when they change
  watch(() => quotes.value, (newQuotes) => {
    localStorage.setItem('quotes', JSON.stringify(newQuotes))
  }, { deep: true })
})
</script>

<template>
  <div class="min-h-screen bg-white transition-colors dark:bg-[#141c26]">
    <div class="mx-auto px-4 py-12 container">
      <div class="mb-12 text-center space-y-4">
        <h1 class="text-4xl text-primary font-bold tracking-tight">
          DollarGill Photography
        </h1>
        <p class="text-lg text-primary/80">
          Professional Photography Services
        </p>
      </div>

      <Card class="mx-auto max-w-3xl w-full border-primary/20">
        <CardHeader class="text-center space-y-4">
          <CardTitle class="text-3xl text-primary font-bold">
            Photography Quote Calculator
          </CardTitle>
          <CardDescription class="text-muted-foreground">
            Customize your photography package
          </CardDescription>
        </CardHeader>

        <CardContent class="space-y-6">
          <Tabs default-value="duration" class="space-y-6">
            <TabsList class="grid grid-cols-3 mb-4 w-full">
              <TabsTrigger value="duration">
                Duration
              </TabsTrigger>
              <TabsTrigger value="details">
                Details
              </TabsTrigger>
              <TabsTrigger value="quote">
                Quote
              </TabsTrigger>
            </TabsList>

            <TabsContent value="duration" class="space-y-4">
              <div class="flex items-center space-x-4">
                <div class="rounded-full bg-primary/10 p-2">
                  <Clock class="h-6 w-6 text-primary" />
                </div>
                <div class="flex-grow">
                  <Label for="hours" class="text-primary">
                    Shoot Duration (hours)
                  </Label>
                  <Slider
                    id="hours"
                    :min="1"
                    :max="5"
                    :step="1"
                    :model-value="hours"
                    class="mt-2"
                    @update:model-value="handleHoursChange"
                  />
                </div>
                <div class="w-20">
                  <Input
                    type="number"
                    :model-value="hours"
                    :min="1"
                    :max="5"
                    class="border-primary/30 focus:border-primary focus:ring-primary/20"
                    @update:model-value="handleHoursChange"
                  />
                </div>
              </div>
            </TabsContent>

            <TabsContent value="details" class="space-y-6">
              <div class="border border-primary/20 rounded-lg bg-primary/5 p-6">
                <h3 class="mb-4 text-primary font-semibold">
                  Included in your package:
                </h3>
                <div class="flex flex-wrap gap-2">
                  <Badge variant="secondary" class="bg-primary/10 text-primary">
                    {{ hours }} hour{{ hours > 1 ? 's' : '' }}
                  </Badge>
                  <Badge variant="secondary" class="bg-primary/10 text-primary">
                    Up to {{ hourlyRates[hours].maxLocations }} locations
                  </Badge>
                  <Badge variant="secondary" class="bg-primary/10 text-primary">
                    {{ hourlyRates[hours].defaultImages }} edited images
                  </Badge>
                </div>
              </div>

              <div class="space-y-6">
                <div class="flex items-center space-x-4">
                  <div class="rounded-full bg-primary/10 p-2">
                    <MapPin class="h-6 w-6 text-primary" />
                  </div>
                  <div class="flex-grow">
                    <Label for="locations" class="text-primary">
                      Number of Locations
                    </Label>
                    <Slider
                      id="locations"
                      :min="1"
                      :max="hourlyRates[hours].maxLocations"
                      :step="1"
                      :model-value="locations"
                      class="mt-2"
                      @update:model-value="setLocations"
                    />
                  </div>
                  <div class="w-20">
                    <Input
                      type="number"
                      :model-value="locations"
                      :min="1"
                      :max="hourlyRates[hours].maxLocations"
                      class="border-primary/30 focus:border-primary focus:ring-primary/20"
                      @update:model-value="setLocations"
                    />
                  </div>
                </div>

                <div class="space-y-4">
                  <div class="flex items-center space-x-4">
                    <div class="rounded-full bg-primary/10 p-2">
                      <Image class="h-6 w-6 text-primary" />
                    </div>
                    <div class="flex-grow">
                      <Label for="additionalImages" class="text-primary">
                        Additional Edited Images (0-100)
                      </Label>
                      <Slider
                        id="additionalImages"
                        :min="0"
                        :max="100"
                        :step="5"
                        :model-value="additionalImagesSlider"
                        class="mt-2"
                        :disabled="extraImages !== null"
                        @update:model-value="handleEditedImagesChange"
                      />
                    </div>
                    <div class="w-20">
                      <Input
                        type="number"
                        :model-value="editedImages - hourlyRates[hours].defaultImages"
                        :min="0"
                        :max="100"
                        :step="5"
                        class="border-primary/30 focus:border-primary focus:ring-primary/20"
                        :disabled="extraImages !== null"
                        @update:model-value="val => handleEditedImagesChange(Math.round(Number(val) / 5) * 5)"
                      />
                    </div>
                  </div>
                  <div class="space-y-2">
                    <Label for="extraImages" class="text-primary">
                      Need more images? Enter a number (100-500)
                    </Label>
                    <Input
                      id="extraImages"
                      type="number"
                      :model-value="extraImages !== null ? extraImages : ''"
                      :min="100"
                      :max="500"
                      class="border-primary/30 focus:border-primary focus:ring-primary/20"
                      placeholder="Enter 100-500"
                      @update:model-value="handleExtraImagesChange"
                    />
                    <p v-if="extraImages === null" class="text-sm text-muted-foreground">
                      You can select images between 0-100 through the slider above.
                    </p>
                  </div>
                </div>
              </div>
            </TabsContent>

            <TabsContent value="quote" class="space-y-6">
              <form class="space-y-4" @submit.prevent="handleSubmit">
                <div class="space-y-2">
                  <Label for="name" class="text-primary">
                    Name
                  </Label>
                  <Input
                    id="name"
                    v-model="name"
                    required
                    class="border-primary/30 focus:border-primary focus:ring-primary/20"
                  />
                </div>
                <div class="space-y-2">
                  <Label for="email" class="text-primary">
                    Email
                  </Label>
                  <Input
                    id="email"
                    v-model="email"
                    type="email"
                    required
                    class="border-primary/30 focus:border-primary focus:ring-primary/20"
                  />
                </div>
                <div class="space-y-2">
                  <Label for="phone" class="text-primary">
                    Phone Number
                  </Label>
                  <Input
                    id="phone"
                    v-model="phone"
                    type="tel"
                    required
                    class="border-primary/30 focus:border-primary focus:ring-primary/20"
                  />
                </div>
                <Button
                  type="submit"
                  class="w-full bg-primary text-white transition-colors hover:bg-primary/90"
                >
                  Get Quote
                </Button>
              </form>
            </TabsContent>
          </Tabs>

          <!-- Active Cost Calculation -->
          <div class="mt-6 border border-primary/20 rounded-lg bg-primary/5 p-4">
            <h3 class="mb-2 text-lg text-primary font-semibold">
              Live Quote Calculation
            </h3>
            <div class="space-y-2">
              <div class="flex justify-between">
                <span class="text-muted-foreground">Base Price ({{ hours }} hour shoot):</span>
                <span class="text-primary font-medium">${{ basePrice.toFixed(2) }}</span>
              </div>
              <div v-if="additionalImages > 0 || extraImages !== null" class="flex justify-between">
                <span class="text-muted-foreground">
                  Additional Images ({{ extraImages !== null ? extraImages : additionalImages }}):
                </span>
                <span class="text-primary font-medium">${{ additionalImagesCost.toFixed(2) }}</span>
              </div>
              <div class="mt-2 flex justify-between border-t border-primary/10 pt-2">
                <span class="text-primary font-semibold">Total (excluding GST):</span>
                <span class="text-primary font-bold">${{ totalCost.toFixed(2) }}</span>
              </div>
            </div>
          </div>

          <div v-if="showSummary" class="mt-8 space-y-6">
            <h3 class="text-xl text-primary font-bold">
              Quote Summary
            </h3>
            <div class="overflow-hidden border border-primary/20 rounded-lg bg-primary/5">
              <Table>
                <TableHeader>
                  <TableRow class="border-primary/20">
                    <TableHead class="text-primary">
                      Item
                    </TableHead>
                    <TableHead class="text-primary">
                      Details
                    </TableHead>
                    <TableHead class="text-right text-primary">
                      Amount
                    </TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  <TableRow class="border-primary/20">
                    <TableCell class="text-primary font-medium">
                      Base Price ({{ hours }} hour shoot)
                    </TableCell>
                    <TableCell class="text-muted-foreground">
                      {{ hours }} hour{{ hours > 1 ? 's' : '' }}, {{ locations }} location{{ locations > 1 ? 's' : '' }},
                      {{ hourlyRates[hours].defaultImages }} images
                    </TableCell>
                    <TableCell class="text-right text-primary">
                      ${{ basePrice.toFixed(2) }}
                    </TableCell>
                  </TableRow>
                  <TableRow v-if="additionalImages > 0 || extraImages !== null" class="border-primary/20">
                    <TableCell class="text-primary font-medium">
                      Additional Image Edits
                    </TableCell>
                    <TableCell class="text-muted-foreground">
                      {{ extraImages !== null ? extraImages : additionalImages }} extra image
                      {{ (extraImages !== null ? extraImages : additionalImages) > 1 ? 's' : '' }} @ $
                      {{ ADDITIONAL_IMAGE_PRICE }} each
                    </TableCell>
                    <TableCell class="text-right text-primary">
                      ${{ additionalImagesCost.toFixed(2) }}
                    </TableCell>
                  </TableRow>
                  <TableRow class="border-primary/20">
                    <TableCell class="text-primary font-medium">
                      Total (excluding GST)
                    </TableCell>
                    <TableCell />
                    <TableCell class="text-right text-primary font-bold">
                      ${{ totalCost.toFixed(2) }}
                    </TableCell>
                  </TableRow>
                </TableBody>
              </Table>
            </div>
            <div class="text-center space-y-2">
              <p class="text-lg text-primary font-semibold">
                Total Estimated Cost (excluding GST)
              </p>
              <p class="text-4xl text-primary font-bold">
                ${{ totalCost.toFixed(2) }}
              </p>
            </div>
          </div>

          <div v-if="quotes.length > 0" class="mt-8 space-y-4">
            <h3 class="text-xl text-primary font-bold">
              Previous Quotes
            </h3>
            <div class="overflow-hidden border border-primary/20 rounded-lg bg-primary/5">
              <Table>
                <TableHeader>
                  <TableRow class="border-primary/20">
                    <TableHead class="text-primary">
                      Date
                    </TableHead>
                    <TableHead class="text-primary">
                      Hours
                    </TableHead>
                    <TableHead class="text-primary">
                      Locations
                    </TableHead>
                    <TableHead class="text-primary">
                      Images
                    </TableHead>
                    <TableHead class="text-right text-primary">
                      Total Cost
                    </TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  <TableRow v-for="(quote, index) in quotes" :key="index" class="border-primary/20">
                    <TableCell class="text-muted-foreground">
                      {{ quote.date }}
                    </TableCell>
                    <TableCell class="text-muted-foreground">
                      {{ quote.hours }}
                    </TableCell>
                    <TableCell class="text-muted-foreground">
                      {{ quote.locations }}
                    </TableCell>
                    <TableCell class="text-muted-foreground">
                      {{ quote.editedImages }}
                    </TableCell>
                    <TableCell class="text-right text-muted-foreground">
                      ${{ quote.totalCost.toFixed(2) }}
                    </TableCell>
                  </TableRow>
                </TableBody>
              </Table>
            </div>
          </div>
        </CardContent>

        <CardFooter class="flex justify-between">
          <Button
            variant="outline"
            class="border-primary/30 bg-primary/5 text-primary transition-colors hover:border-primary/50 hover:bg-primary/10"
            @click="resetForm"
          >
            Reset
          </Button>
        </CardFooter>
      </Card>

      <footer class="mt-8 text-center text-sm text-muted-foreground">
        Copyright DollarGill 2024. All Rights Reserved.
      </footer>
    </div>

    <Dialog :open="showQuotaExhaustedDialog" @update:open="showQuotaExhaustedDialog = $event">
      <DialogContent class="border-primary/20 bg-white dark:bg-[#141c26]">
        <DialogHeader>
          <DialogTitle class="text-primary">
            Quote Limit Reached
          </DialogTitle>
          <DialogDescription class="text-muted-foreground">
            You have exhausted your free quote generator quota for today. Either send an email to DollarGill or change
            your contact details to generate more quotes.
          </DialogDescription>
        </DialogHeader>
        <div class="flex justify-end space-x-2">
          <Button
            variant="outline"
            class="border-primary/50 text-primary transition-colors hover:bg-primary/10"
            @click="showQuotaExhaustedDialog = false"
          >
            Close
          </Button>
          <Button
            class="bg-primary text-white transition-colors hover:bg-primary/90"
            @click="window.location.href = 'mailto:contact@dollargill.com'"
          >
            Email DollarGill
          </Button>
        </div>
      </DialogContent>
    </Dialog>
  </div>
</template>

  <style>
  /* UnoCSS will handle most styling, but we can add custom styles here */
:root {
  --primary: #6366f1;
  --primary-foreground: #ffffff;
  --muted-foreground: #64748b;
}

.dark {
  --primary: #818cf8;
  --primary-foreground: #ffffff;
  --muted-foreground: #94a3b8;
}

/* Add animation for background if needed */
@keyframes gradientAnimation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
