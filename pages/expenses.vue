<script setup>
import {
  ArrowUpRight,
  Calendar,
  Car,
  CreditCard,
  DollarSign,
  Film,
  Home,
  MoreHorizontal,
  PieChart,
  Plus,
  Search,
  ShoppingBag,
  Utensils,
  Wallet,
  Zap,
} from 'lucide-vue-next'
import { computed, reactive, ref } from 'vue'

// Define expense type
// type Expense = {
//   id: string
//   amount: number
//   description: string
//   category: string
//   date: string
// }

// Define categories with colors and icons
const categories = [
  { name: 'Domains', color: 'bg-blue-500/80', icon: Utensils },
  { name: 'Workers', color: 'bg-green-500/80', icon: Car },
  { name: 'Utilities', color: 'bg-pink-500/80', icon: ShoppingBag },
  { name: 'Services', color: 'bg-yellow-500/80', icon: Zap },
  { name: 'Hosting', color: 'bg-orange-500/80', icon: Film },
  { name: 'Other', color: 'bg-purple-500/80', icon: Home },
]

// State for expenses
const expenses = ref([])

// State for new expense
const newExpense = reactive({
  amount: '',
  description: '',
  category: 'Food',
  date: new Date().toISOString().split('T')[0],
})

// State for search and filter
const searchTerm = ref('')
const filterCategory = ref('All')
const isAddExpenseOpen = ref(false)
const activeTab = ref('all')
const activeDropdown = ref(null)

// Toggle dropdown menu
function toggleDropdown(id) {
  if (activeDropdown.value === id) {
    activeDropdown.value = null
  }
  else {
    activeDropdown.value = id
  }
}

// Calculate total expenses
const totalExpenses = computed(() => {
  return expenses.value.reduce((sum, expense) => sum + expense.amount, 0)
})

// Calculate this month's expenses
const thisMonthTotal = computed(() => {
  return expenses.value
    .filter(e => new Date(e.date).getMonth() === new Date().getMonth())
    .reduce((sum, e) => sum + e.amount, 0)
})

const thisMonthCount = computed(() => {
  return expenses.value.filter(e => new Date(e.date).getMonth() === new Date().getMonth()).length
})

// Filter expenses based on search, category, and active tab
const filteredExpenses = computed(() => {
  return expenses.value.filter((expense) => {
    const matchesSearch = expense.description.toLowerCase().includes(searchTerm.value.toLowerCase())
    const matchesCategory = filterCategory.value === 'All' || expense.category === filterCategory.value

    // Filter by time period if needed
    const expenseDate = new Date(expense.date)
    const today = new Date()
    const isToday = expenseDate.toDateString() === today.toDateString()

    const startOfWeek = new Date(today)
    startOfWeek.setDate(today.getDate() - today.getDay())
    startOfWeek.setHours(0, 0, 0, 0)
    const isThisWeek = expenseDate >= startOfWeek

    const isThisMonth
      = expenseDate.getMonth() === new Date().getMonth()
        && expenseDate.getFullYear() === new Date().getFullYear()

    let matchesTab = true
    if (activeTab.value === 'today')
      matchesTab = isToday
    else if (activeTab.value === 'week')
      matchesTab = isThisWeek
    else if (activeTab.value === 'month')
      matchesTab = isThisMonth

    return matchesSearch && matchesCategory && matchesTab
  })
})

// Group expenses by category for summary
const expensesByCategory = computed(() => {
  return categories
    .map((category) => {
      const total = expenses.value
        .filter(expense => expense.category === category.name)
        .reduce((sum, expense) => sum + expense.amount, 0)
      return {
        ...category,
        total,
      }
    })
    .sort((a, b) => b.total - a.total)
})

// Handle adding a new expense
function handleAddExpense() {
  if (!newExpense.amount || !newExpense.description)
    return

  const expense = {
    id: Date.now().toString(),
    amount: Number.parseFloat(newExpense.amount),
    description: newExpense.description,
    category: newExpense.category,
    date: newExpense.date,
  }

  expenses.value = [expense, ...expenses.value]

  // Reset form
  newExpense.amount = ''
  newExpense.description = ''
  newExpense.category = 'Food'
  newExpense.date = new Date().toISOString().split('T')[0]

  isAddExpenseOpen.value = false
}

// Handle deleting an expense
function handleDeleteExpense(id) {
  expenses.value = expenses.value.filter(expense => expense.id !== id)
  activeDropdown.value = null
}

// Get category color
function getCategoryColor(categoryName) {
  const category = categories.find(c => c.name === categoryName)
  return category ? category.color : ' bg-white dark:bg-[#141c26]'
}

// Format date
function formatDate(dateString) {
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
}

// Get category icon
function getCategoryIcon(categoryName) {
  const category = categories.find(c => c.name === categoryName)
  return category ? category.icon : CreditCard
}
</script>

<template>
  <div class="min-h-screen flex flex-col bg-white dark:bg-[#141c26]">
    <!-- Header -->
    <header class="bg-white- sticky top-0 z-10 border-b border-white/20 backdrop-blur-xl dark:bg-[#141c26]">
      <div class="mx-auto h-16 flex items-center justify-between px-4 container">
        <div class="flex items-center gap-2">
          <Wallet class="text-muted-forground h-6 w-6" />
          <h1 class="text-xl text-primary font-semibold">
            ExpenseTracker
          </h1>
        </div>
        <div class="flex items-center gap-3">
          <button class="text-muted-forground rounded-full bg-white p-2 dark:bg-[#141c26] hover:bg-white/20">
            <Search class="h-4 w-4" />
            <span class="sr-only">Search</span>
          </button>
          <button
            class="text-muted-forground flex items-center gap-1 border-white/30 rounded-full bg-white px-3 py-1 backdrop-blur-md dark:bg-[#141c26] hover:bg-primary/50"
            @click="isAddExpenseOpen = true"
          >
            <Plus class="h-4 w-4" />
            <span class="hidden sm:inline">Add Expense</span>
          </button>
          <div
            class="text-muted-forground h-9 w-9 flex items-center justify-center border-2 border-white/30 rounded-full bg-white backdrop-blur-sm dark:bg-[#141c26] hover:bg-primary/50"
          >
            U
          </div>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="mx-auto flex-1 px-4 py-6 container">
      <div class="grid gap-6">
        <!-- Summary Cards -->
        <div class="grid grid-cols-1 gap-4 md:grid-cols-3">
          <div
            class="rounded-lgbg-white overflow-hidden border-white/20 shadow-xl backdrop-blur-xl dark:bg-[#141c26] hover:bg-primary/50"
          >
            <div class="p-4 pb-2">
              <div class="text-muted-forground text-sm">
                Total Expenses
              </div>
              <div class="text-muted-forground text-3xl font-bold">
                ${{ totalExpenses.toFixed(2) }}
              </div>
            </div>
            <div class="px-4 pb-2">
              <div class="text-muted-forground flex items-center text-xs">
                <ArrowUpRight class="mr-1 h-3 w-3 text-green-300" />
                <span class="text-green-300 font-medium">12%</span> from last month
              </div>
            </div>
            <div class="h-2 from-blue-500/80 to-purple-500/80 bg-gradient-to-r" />
          </div>

          <div
            class="border-white/20 rounded-lg bg-white shadow-xl backdrop-blur-xl dark:bg-[#141c26] hover:bg-primary/50"
          >
            <div class="p-4 pb-2">
              <div class="text-muted-forground text-sm">
                Top Category
              </div>
              <div class="text-muted-forground flex items-center gap-2">
                <div :class="`w-3 h-3 rounded-full ${expensesByCategory[0]?.color || 'bg-gray-500/80'}`" />
                {{ expensesByCategory[0]?.name || 'None' }}
              </div>
            </div>
            <div class="px-4 pb-2">
              <div class="text-muted-forground text-2xl font-bold">
                ${{ expensesByCategory[0]?.total.toFixed(2) || '0.00' }}
              </div>
              <div class="text-muted-forground mt-1 text-xs">
                {{ Math.round((expensesByCategory[0]?.total / totalExpenses) * 100) || 0 }}% of total expenses
              </div>
            </div>
            <div class="h-2 from-green-500/80 to-emerald-500/80 bg-gradient-to-r" />
          </div>

          <div
            class="border-white/20 rounded-lg bg-white shadow-xl backdrop-blur-xl dark:bg-[#141c26] hover:bg-primary/50"
          >
            <div class="p-4 pb-2">
              <div class="text-muted-forground text-sm">
                This Month
              </div>
              <div class="text-muted-forground text-3xl font-bold">
                ${{ thisMonthTotal.toFixed(2) }}
              </div>
            </div>
            <div class="px-4 pb-2">
              <div class="text-muted-forground text-xs">
                {{ thisMonthCount }} transactions
              </div>
            </div>
            <div class="h-2 from-orange-500/80 to-[#141c26] bg-gradient-to-r" />
          </div>
        </div>

        <!-- Category Distribution -->
        <div
          class="border-white/20 rounded-lg bg-white shadow-xl backdrop-blur-xl dark:bg-[#141c26]"
        >
          <div class="border-b border-white/10 p-4">
            <div class="text-muted-forground text-lg">
              Expense Distribution
            </div>
          </div>
          <div class="p-4">
            <div class="grid grid-cols-1 gap-4 md:grid-cols-3 sm:grid-cols-2">
              <div
                v-for="category in expensesByCategory.filter(c => c.total > 0)"
                :key="category.name"
                class="flex items-center gap-3 rounded-xl bg-white/5 p-3 transition-colors hover:bg-white/10"
              >
                <div
                  :class="`w-10 h-10 rounded-full ${category.color} flex items-center justify-center text-muted-forground shadow-lg`"
                >
                  <component :is="category.icon" class="h-4 w-4" />
                </div>
                <div>
                  <div class="text-muted-forground font-medium">
                    {{ category.name }}
                  </div>
                  <div class="text-muted-forground text-sm">
                    ${{ category.total.toFixed(2) }}
                  </div>
                </div>
                <div class="ml-auto">
                  <div class="text-muted-forground text-sm font-medium">
                    {{ Math.round((category.total / totalExpenses) * 100) }}%
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Expense List -->
        <div
          class="hover:bg-primaryshadow-xl border-white/20 rounded-lg bg-white backdrop-blur-xl dark:bg-[#141c26]"
        >
          <div class="border-b border-white/10 p-4 pb-3">
            <div class="flex items-center justify-between">
              <div class="text-muted-forground text-lg">
                Recent Expenses
              </div>
              <div class="flex items-center gap-2">
                <div class="relative">
                  <Search class="text-muted-forground absolute left-2.5 top-2.5 h-4 w-4" />
                  <input
                    v-model="searchTerm"
                    type="search"
                    placeholder="Search expenses..."
                    class="text-muted-forground placeholder:text-muted-forground h-9 w-[200px] border-white/20 rounded-md bg-white/10 pl-8"
                  >
                </div>
                <select
                  v-model="filterCategory"
                  class="text-muted-forground h-9 w-[130px] border-white/20 rounded-md bg-white px-2 dark:bg-[#141c26]"
                >
                  <option value="All">
                    All Categories
                  </option>
                  <option v-for="category in categories" :key="category.name" :value="category.name">
                    {{ category.name }}
                  </option>
                </select>
              </div>
            </div>
            <div class="mt-2">
              <div
                class="grid grid-cols-4 w-full rounded-md bg-white sm:w-[400px] dark:bg-[#141c26]"
              >
                <button
                  v-for="tab in ['all', 'today', 'week', 'month']"
                  :key="tab"
                  class="text-muted-forground py-1.5 text-sm" :class="[
                    activeTab === tab ? 'bg-primary/50' : '',
                  ]"
                  @click="activeTab = tab"
                >
                  {{ tab === 'all' ? 'All' : tab === 'today' ? 'Today' : tab === 'week' ? 'This Week' : 'This Month' }}
                </button>
              </div>
            </div>
          </div>
          <div class="divide-y divide-white/10">
            <template v-if="filteredExpenses.length > 0">
              <div
                v-for="expense in filteredExpenses"
                :key="expense.id"
                class="flex items-center justify-between p-4 transition-colors hover:bg-white/5"
              >
                <div class="flex items-center gap-3">
                  <div
                    :class="`w-10 h-10 rounded-full ${getCategoryColor(expense.category)} flex items-center justify-center text-muted-forground shadow-lg`"
                  >
                    <component :is="getCategoryIcon(expense.category)" class="h-4 w-4" />
                  </div>
                  <div>
                    <div class="text-muted-forground font-medium">
                      {{ expense.description }}
                    </div>
                    <div class="text-muted-forground flex items-center gap-2 text-sm">
                      <span class="text-muted-forground rounded-full bg-white/10 px-2 py-0.5 text-xs">
                        {{ expense.category }}
                      </span>
                      <span>•</span>
                      <div class="flex items-center gap-1">
                        <Calendar class="h-3 w-3" />
                        {{ formatDate(expense.date) }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="flex items-center gap-4">
                  <div class="text-muted-forground text-lg font-medium">
                    ${{ expense.amount.toFixed(2) }}
                  </div>
                  <div class="relative">
                    <button
                      class="text-muted-forground h-8 w-8 flex items-center justify-center rounded-full hover:bg-white/10"
                      @click="toggleDropdown(expense.id)"
                    >
                      <MoreHorizontal class="h-4 w-4" />
                    </button>
                    <div
                      v-if="activeDropdown === expense.id"
                      class="absolute right-0 z-10 mt-1 min-w-[120px] rounded-md bg-white py-1 shadow-lg backdrop-blur-xl dark:bg-[#141c26] hover:bg-primary"
                    >
                      <button class="w-full px-3 py-1 text-left hover:bg-white/20">
                        Edit
                      </button>
                      <button
                        class="w-full px-3 py-1 text-left text-red-500 hover:bg-white/20"
                        @click="handleDeleteExpense(expense.id)"
                      >
                        Delete
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </template> <div v-else class="py-12 text-center text-white/70">
              <PieChart class="mx-auto mb-3 h-12 w-12 text-white/30" />
              <p>No expenses found.</p>
              <p class="text-sm">
                Add a new expense or adjust your filters.
              </p>
              <button
                class="mx-auto mt-4 flex items-center justify-center border border-white/20 rounded-md bg-white/10 px-4 py-2 text-white hover:bg-white/20"
                @click="isAddExpenseOpen = true"
              >
                <Plus class="mr-2 h-4 w-4" />
                Add Expense
              </button>
            </div>
          </div>
          <div v-if="filteredExpenses.length > 0" class="flex justify-between border-t border-white/10 px-4 py-4">
            <div class="text-muted-forground text-sm">
              Showing {{ filteredExpenses.length }} of {{ expenses.length }} expenses
            </div>
            <button
              class="text-muted-forground border border-white/20 rounded-md bg-white/10 px-3 py-1 text-sm hover:bg-white/20"
            >
              View All
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- Add Expense Dialog -->
    <Teleport to="body">
      <div v-if="isAddExpenseOpen" class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm">
        <div
          class="mx-4 max-w-[425px] w-full border border-white/30 rounded-lg bg-white/80 backdrop-blur-xl dark:bg-[#141c26]"
        >
          <div class="border-b border-white/20 p-4">
            <h2 class="text-lg font-semibold">
              Add New Expense
            </h2>
          </div>
          <div class="p-4">
            <div class="grid gap-4">
              <div class="grid gap-2">
                <label for="amount" class="text-sm font-medium">Amount</label>
                <div class="relative">
                  <DollarSign class="absolute left-3 top-2.5 h-4 w-4 text-gray-500" />
                  <input
                    id="amount"
                    v-model="newExpense.amount"
                    type="number"
                    placeholder="0.00"
                    class="h-10 w-full border border-white/30 rounded-md bg-white px-3 pl-9 hover:border-2 hover:border-primary dark:bg-[#141c26]"
                  >
                </div>
              </div>
              <div class="grid gap-2">
                <label for="description" class="text-sm font-medium">Description</label>
                <input
                  id="description"
                  v-model="newExpense.description"
                  placeholder="What did you spend on?"
                  class="h-10 w-full border border-white/30 rounded-md bg-white px-3 hover:border-2 hover:border-primary dark:bg-[#141c26]"
                >
              </div>
              <div class="grid gap-2">
                <label for="category" class="text-sm font-medium">Category</label>
                <select
                  id="category"
                  v-model="newExpense.category"
                  class="h-10 w-full border border-white/30 rounded-md bg-white px-3 hover:border-2 hover:border-primary dark:bg-[#141c26]"
                >
                  <option v-for="category in categories" :key="category.name" :value="category.name">
                    {{ category.name }}
                  </option>
                </select>
              </div>
              <div class="grid gap-2">
                <label for="date" class="text-sm font-medium">Date</label>
                <div class="relative">
                  <Calendar class="absolute left-3 top-2.5 h-4 w-4 text-gray-500" />
                  <input
                    id="date"
                    v-model="newExpense.date"
                    type="date"
                    class="h-10 w-full border border-white/30 rounded-md bg-white px-3 pl-9 hover:border-2 hover:border-primary dark:bg-[#141c26]"
                  >
                </div>
              </div>
            </div>
          </div>
          <div class="flex justify-end gap-2 border-t border-white/20 p-4">
            <button
              class="text-forground rounded-md bg-white px-3 py-2 hover:border-2 hover:border-primary dark:bg-[#141c26]"
              @click="isAddExpenseOpen = false"
            >
              Cancel
            </button>
            <button
              class="border-0 rounded-md bg-primary px-3 py-2 text-secondary"
              @click="handleAddExpense"
            >
              Add Expense
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<style>
/* Tailwind Preflight (minimal reset) */
*,
::before,
::after {
  box-sizing: border-box;
  border-width: 0;
  border-style: solid;
}

html {
  line-height: 1.5;
  -webkit-text-size-adjust: 100%;
  -moz-tab-size: 4;
  tab-size: 4;
  font-family: ui-sans-serif, system-ui, sans-serif;
}

body {
  margin: 0;
  line-height: inherit;
}

hr {
  height: 0;
  color: inherit;
  border-top-width: 1px;
}

button,
input,
optgroup,
select,
textarea {
  font-family: inherit;
  font-size: 100%;
  font-weight: inherit;
  line-height: inherit;
  color: inherit;
  margin: 0;
  padding: 0;
}

button,
select {
  text-transform: none;
}

/* Custom styles for glassmorphism */
.backdrop-blur-xl {
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}

.backdrop-blur-md {
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.backdrop-blur-sm {
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}
</style>
