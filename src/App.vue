<script setup>
import { ref, computed, watch } from 'vue'

const STORAGE_KEY = 'school-demo-v1'
const CLASSES = ['Class 6', 'Class 7', 'Class 8', 'Class 9', 'Class 10']
const FEE = 4500

const seed = () => [
  ['Ayesha Khan', 'Class 6', 'Imran Khan', true],
  ['Hamza Ali', 'Class 6', 'Ali Raza', false],
  ['Fatima Noor', 'Class 7', 'Noor Ahmed', true],
  ['Usman Tariq', 'Class 7', 'Tariq Mehmood', true],
  ['Zainab Iqbal', 'Class 8', 'Iqbal Hussain', false],
  ['Bilal Ahmed', 'Class 8', 'Ahmed Yar', true],
  ['Maryam Saeed', 'Class 9', 'Saeed Anwar', true],
  ['Hassan Raza', 'Class 9', 'Raza Abbas', false],
  ['Sana Javed', 'Class 9', 'Javed Iqbal', true],
  ['Ahmad Farooq', 'Class 10', 'Farooq Malik', true],
  ['Hira Aslam', 'Class 10', 'Aslam Butt', false],
  ['Daniyal Shah', 'Class 10', 'Shah Nawaz', true],
].map(([name, cls, guardian, paid], i) => ({
  id: i + 1,
  roll: 100 + i + 1,
  name,
  cls,
  guardian,
  paid,
  status: i % 5 === 3 ? 'absent' : 'present',
}))

function load() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (raw) return JSON.parse(raw)
  } catch (e) {}
  return seed()
}

const students = ref(load())
watch(students, (v) => { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(v)) } catch (e) {} }, { deep: true })

const tab = ref('dashboard')
const tabs = [['dashboard', 'Dashboard'], ['students', 'Students'], ['attendance', 'Attendance'], ['fees', 'Fees']]

const search = ref('')
const classFilter = ref('All')
const filtered = computed(() =>
  students.value.filter((s) =>
    (classFilter.value === 'All' || s.cls === classFilter.value) &&
    (s.name.toLowerCase().includes(search.value.toLowerCase()) || String(s.roll).includes(search.value)),
  ),
)

const blank = () => ({ name: '', cls: CLASSES[0], guardian: '' })
const form = ref(blank())
const showForm = ref(false)

function addStudent() {
  if (!form.value.name.trim()) return
  const nextId = Math.max(0, ...students.value.map((s) => s.id)) + 1
  students.value.push({ id: nextId, roll: 100 + nextId, ...form.value, name: form.value.name.trim(), paid: false, status: 'present' })
  form.value = blank()
  showForm.value = false
}
function removeStudent(id) { students.value = students.value.filter((s) => s.id !== id) }
function resetDemo() { students.value = seed() }

const stats = computed(() => {
  const total = students.value.length
  const present = students.value.filter((s) => s.status === 'present').length
  const paid = students.value.filter((s) => s.paid).length
  return {
    total,
    present,
    attendance: total ? Math.round((present / total) * 100) : 0,
    collected: paid * FEE,
    pending: (total - paid) * FEE,
  }
})
const byClass = computed(() => CLASSES.map((c) => ({ cls: c, count: students.value.filter((s) => s.cls === c).length })))
const maxCount = computed(() => Math.max(1, ...byClass.value.map((c) => c.count)))
const pendingList = computed(() => students.value.filter((s) => !s.paid))
const today = new Date().toLocaleDateString('en-GB', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' })
const money = (n) => 'Rs. ' + n.toLocaleString('en-PK')
const statusStyle = { present: 'bg-emerald-100 text-emerald-700', absent: 'bg-red-100 text-red-700', leave: 'bg-amber-100 text-amber-700' }
</script>

<template>
  <div class="bg-amber-100 px-4 py-2 text-center text-xs text-amber-900">
    Demo project with sample data (saved in your browser only) Â·
    <a href="https://luqmanarshad.github.io/" class="font-semibold underline">Back to portfolio</a>
  </div>

  <div class="mx-auto max-w-6xl px-4 py-6">
    <header class="flex flex-wrap items-center justify-between gap-3">
      <div>
        <h1 class="text-2xl font-extrabold text-slate-900">Green Valley School</h1>
        <p class="text-sm text-slate-500">School Management System Â· {{ today }}</p>
      </div>
      <button class="rounded-lg border border-slate-300 bg-white px-3 py-2 text-xs font-medium hover:bg-slate-50" @click="resetDemo">Reset demo data</button>
    </header>

    <nav class="mt-5 flex gap-1 overflow-x-auto rounded-xl bg-white p-1 shadow-sm" aria-label="Sections">
      <button v-for="[key, label] in tabs" :key="key" class="whitespace-nowrap rounded-lg px-4 py-2 text-sm font-semibold transition"
        :class="tab === key ? 'bg-indigo-600 text-white' : 'text-slate-600 hover:bg-slate-100'" @click="tab = key">{{ label }}</button>
    </nav>

    <!-- Dashboard -->
    <section v-if="tab === 'dashboard'" class="mt-5 space-y-5">
      <div class="grid grid-cols-2 gap-4 lg:grid-cols-4">
        <div class="rounded-2xl bg-white p-5 shadow-sm"><p class="text-xs text-slate-500">Total students</p><p class="mt-1 text-3xl font-extrabold">{{ stats.total }}</p></div>
        <div class="rounded-2xl bg-white p-5 shadow-sm"><p class="text-xs text-slate-500">Present today</p><p class="mt-1 text-3xl font-extrabold text-emerald-600">{{ stats.attendance }}%</p><p class="text-xs text-slate-400">{{ stats.present }} of {{ stats.total }}</p></div>
        <div class="rounded-2xl bg-white p-5 shadow-sm"><p class="text-xs text-slate-500">Fees collected</p><p class="mt-1 text-2xl font-extrabold text-indigo-600">{{ money(stats.collected) }}</p></div>
        <div class="rounded-2xl bg-white p-5 shadow-sm"><p class="text-xs text-slate-500">Fees pending</p><p class="mt-1 text-2xl font-extrabold text-red-600">{{ money(stats.pending) }}</p></div>
      </div>
      <div class="rounded-2xl bg-white p-5 shadow-sm">
        <h2 class="font-bold">Students per class</h2>
        <div class="mt-4 space-y-3">
          <div v-for="c in byClass" :key="c.cls" class="flex items-center gap-3 text-sm">
            <span class="w-16 text-slate-500">{{ c.cls }}</span>
            <div class="h-3 flex-1 rounded-full bg-slate-100"><div class="h-3 rounded-full bg-indigo-500 transition-all" :style="{ width: (c.count / maxCount) * 100 + '%' }"></div></div>
            <span class="w-6 text-right font-semibold">{{ c.count }}</span>
          </div>
        </div>
      </div>
    </section>

    <!-- Students -->
    <section v-if="tab === 'students'" class="mt-5 rounded-2xl bg-white p-5 shadow-sm">
      <div class="flex flex-wrap gap-3">
        <input v-model="search" type="search" placeholder="Search name or roll no." aria-label="Search students" class="min-w-[200px] flex-1 rounded-lg border border-slate-300 px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
        <select v-model="classFilter" aria-label="Filter by class" class="rounded-lg border border-slate-300 px-3 py-2 text-sm">
          <option>All</option><option v-for="c in CLASSES" :key="c">{{ c }}</option>
        </select>
        <button class="rounded-lg bg-indigo-600 px-4 py-2 text-sm font-semibold text-white hover:bg-indigo-500" @click="showForm = !showForm">{{ showForm ? 'Close' : '+ Add student' }}</button>
      </div>

      <form v-if="showForm" class="mt-4 grid gap-3 rounded-xl bg-slate-50 p-4 sm:grid-cols-4" @submit.prevent="addStudent">
        <input v-model="form.name" required placeholder="Student name" aria-label="Student name" class="rounded-lg border border-slate-300 px-3 py-2 text-sm" />
        <select v-model="form.cls" aria-label="Class" class="rounded-lg border border-slate-300 px-3 py-2 text-sm"><option v-for="c in CLASSES" :key="c">{{ c }}</option></select>
        <input v-model="form.guardian" placeholder="Guardian name" aria-label="Guardian name" class="rounded-lg border border-slate-300 px-3 py-2 text-sm" />
        <button class="rounded-lg bg-emerald-600 px-4 py-2 text-sm font-semibold text-white hover:bg-emerald-500">Save</button>
      </form>

      <div class="mt-4 overflow-x-auto">
        <table class="w-full text-left text-sm">
          <thead class="text-xs uppercase text-slate-500"><tr><th class="py-2 pr-3">Roll</th><th class="pr-3">Name</th><th class="pr-3">Class</th><th class="pr-3">Guardian</th><th class="pr-3">Fee</th><th></th></tr></thead>
          <tbody>
            <tr v-for="s in filtered" :key="s.id" class="border-t border-slate-100">
              <td class="py-2.5 pr-3 font-mono text-xs">{{ s.roll }}</td>
              <td class="pr-3 font-semibold">{{ s.name }}</td>
              <td class="pr-3">{{ s.cls }}</td>
              <td class="pr-3 text-slate-500">{{ s.guardian || 'â€”' }}</td>
              <td class="pr-3"><span class="rounded-full px-2.5 py-0.5 text-xs font-semibold" :class="s.paid ? 'bg-emerald-100 text-emerald-700' : 'bg-red-100 text-red-700'">{{ s.paid ? 'Paid' : 'Pending' }}</span></td>
              <td class="text-right"><button class="text-xs font-medium text-red-600 hover:underline" @click="removeStudent(s.id)">Delete</button></td>
            </tr>
            <tr v-if="!filtered.length"><td colspan="6" class="py-8 text-center text-slate-400">No students found.</td></tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- Attendance -->
    <section v-if="tab === 'attendance'" class="mt-5 rounded-2xl bg-white p-5 shadow-sm">
      <div class="flex flex-wrap items-center justify-between gap-3">
        <h2 class="font-bold">Mark attendance Â· {{ today }}</h2>
        <select v-model="classFilter" aria-label="Filter by class" class="rounded-lg border border-slate-300 px-3 py-2 text-sm"><option>All</option><option v-for="c in CLASSES" :key="c">{{ c }}</option></select>
      </div>
      <ul class="mt-4 divide-y divide-slate-100">
        <li v-for="s in filtered" :key="s.id" class="flex flex-wrap items-center justify-between gap-3 py-3">
          <div><p class="font-semibold">{{ s.name }}</p><p class="text-xs text-slate-500">{{ s.cls }} Â· Roll {{ s.roll }}</p></div>
          <div class="flex gap-1" role="group" :aria-label="'Attendance for ' + s.name">
            <button v-for="opt in ['present', 'absent', 'leave']" :key="opt" class="rounded-lg px-3 py-1.5 text-xs font-semibold capitalize transition"
              :class="s.status === opt ? statusStyle[opt] + ' ring-1 ring-current' : 'bg-slate-100 text-slate-500 hover:bg-slate-200'" @click="s.status = opt">{{ opt }}</button>
          </div>
        </li>
      </ul>
    </section>

    <!-- Fees -->
    <section v-if="tab === 'fees'" class="mt-5 rounded-2xl bg-white p-5 shadow-sm">
      <div class="flex flex-wrap items-center justify-between gap-2">
        <h2 class="font-bold">Pending fees ({{ pendingList.length }})</h2>
        <p class="text-sm text-slate-500">Monthly fee: {{ money(FEE) }} per student</p>
      </div>
      <ul class="mt-4 divide-y divide-slate-100">
        <li v-for="s in pendingList" :key="s.id" class="flex items-center justify-between gap-3 py-3">
          <div><p class="font-semibold">{{ s.name }}</p><p class="text-xs text-slate-500">{{ s.cls }} Â· Guardian: {{ s.guardian }}</p></div>
          <div class="flex items-center gap-3"><span class="text-sm font-semibold text-red-600">{{ money(FEE) }}</span>
            <button class="rounded-lg bg-emerald-600 px-3 py-1.5 text-xs font-semibold text-white hover:bg-emerald-500" @click="s.paid = true">Mark paid</button></div>
        </li>
        <li v-if="!pendingList.length" class="py-10 text-center text-slate-400">All fees are collected. ðŸŽ‰</li>
      </ul>
    </section>
  </div>
</template>
