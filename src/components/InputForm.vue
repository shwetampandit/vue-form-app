  <script setup>
  // Core imports
  import { ref, reactive, computed } from 'vue'
  import InputGroup from './InputGroup.vue'
  import '@vueform/multiselect/themes/default.css'
  import Multiselect from '@vueform/multiselect'
  import LoadingSpinner from './common/LoadingSpinner.vue'

  defineProps({
    msg: String,
  })

  // State management
  const count = ref(0)
  const rawDate = ref('')
  const fileName = ref('')
  const error = ref('')
  const isSubmitting = ref(false)
  const submitSuccess = ref(false)
  const submitError = ref('')
  const submittedForms = ref([])

  // Form validation state
  const formErrors = reactive({
    groups: [],
    birthDate: '',
    gender: '',
    language: '',
    cities: '',
    file: ''
  })

  // Auth state
  const token = ref(localStorage.getItem('jwt_token') || '')
  const isAuthenticated = computed(() => !!token.value)

  // Form options
  const gender = ref('')
  const genderOptions = ['Male', 'Female', 'Other']
  const language = ref('')
  const languageOptions = ["Angular", "Vuejs", "ReactJs"]
  const selectedCities = ref([])
  const cities = [
    { value: 'Mumbai', label: 'Mumbai' },
    { value: 'Delhi', label: 'Delhi' },
    { value: 'Bengaluru', label: 'Bengaluru' },
    { value: 'Pune', label: 'Pune' },
    { value: 'Hyderabad', label: 'Hyderabad' },
    { value: 'Ahmedabad', label: 'Ahmedabad' },
    { value: 'Kolkata', label: 'Kolkata' },
    { value: 'Chennai', label: 'Chennai' },
    { value: 'Jaipur', label: 'Jaipur' },
    { value: 'Surat', label: 'Surat' }
  ]

  // Format date for display
  const formattedDate = computed(() => {
    if (!rawDate.value) return ''
    const date = new Date(rawDate.value)
    return date.toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    })
  })

  // Form validation
  function validateForm() {
    let isValid = true
    formErrors.groups = []
    // Validate groups
    groups.forEach((group, index) => {
      const groupErrors = {}
      if (!group.name.trim()) {
        groupErrors.name = 'Name is required'
        isValid = false
      }
      if (!group.email.trim()) {
        groupErrors.email = 'Email is required'
        isValid = false
      } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(group.email)) {
        groupErrors.email = 'Invalid email format'
        isValid = false
      }
      if (!group.mobile.trim()) {
        groupErrors.mobile = 'Mobile number is required'
        isValid = false
      } else if (!/^\d{10}$/.test(group.mobile)) {
        groupErrors.mobile = 'Mobile number must be 10 digits'
        isValid = false
      }
      if (Object.keys(groupErrors).length > 0) {
        formErrors.groups[index] = groupErrors
      }
    })

    // Validate other fields
    if (!rawDate.value) {
      formErrors.birthDate = 'Birth date is required'
      isValid = false
    } else {
      const date = new Date(rawDate.value)
      const today = new Date()
      if (date > today) {
        formErrors.birthDate = 'Birth date cannot be in the future'
        isValid = false
      }
    }

    if (!gender.value) formErrors.gender = 'Gender is required'
    if (!language.value) formErrors.language = 'Language is required'
    if (!selectedCities.value.length) formErrors.cities = 'At least one city must be selected'
    if (!fileName.value) formErrors.file = 'File is required'

    return isValid
  }

  // File handling
  function handleFileChange(event) {
    const file = event.target.files[0]
    if (!file) {
      fileName.value = ''
      error.value = ''
      formErrors.file = ''
      return
    }

    const allowedTypes = ['image/jpeg', 'application/pdf']
    const maxSize = 5 * 1024 * 1024 // 5MB

    if (!allowedTypes.includes(file.type)) {
      error.value = 'Only JPEG and PDF files are allowed.'
      formErrors.file = 'Only JPEG and PDF files are allowed.'
      fileName.value = ''
      return
    }

    if (file.size > maxSize) {
      error.value = 'File size must be less than 5MB'
      formErrors.file = 'File size must be less than 5MB'
      fileName.value = ''
      return
    }

    error.value = ''
    formErrors.file = ''
    fileName.value = file.name
  }

  // Form groups management
  const groups = reactive([
    { id: Date.now(), name: '', email: '', mobile: '' },
    { id: Date.now() + 1, name: '', email: '', mobile: '' }
  ])

  function addGroup() {
    groups.push({
      id: Date.now() + Math.random(),
      name: '',
      email: '',
      mobile: ''
    })
  }

  function removeGroup(index) {
    if (groups.length > 2) {
      groups.splice(index, 1)
    }
  }

  // API simulation
  const simulateApiCall = (data) => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (Math.random() > 0.1) {
          resolve({ success: true, data: { id: Date.now(), ...data } })
        } else {
          reject(new Error('Simulated API error'))
        }
      }, 1500)
    })
  }

  // Form submission
  async function handleSubmit(event) {
    event.preventDefault()
    
    if (!isAuthenticated.value) {
      submitError.value = 'Please login first'
      return
    }

    const formDataCopy = {
      groups: JSON.parse(JSON.stringify(groups)),
      birthDate: rawDate.value,
      gender: gender.value,
      language: language.value,
      cities: selectedCities.value,
      fileName: fileName.value
    }

    if (!validateForm()) {
      submitError.value = 'Please fix the form errors before submitting'
      return
    }

    isSubmitting.value = true
    submitError.value = ''
    submitSuccess.value = false

    try {
      const formData = {
        id: Date.now(),
        submittedAt: new Date().toISOString(),
        groups: formDataCopy.groups.map(group => ({
          name: group.name,
          email: group.email,
          mobile: group.mobile
        })),
        birthDate: formDataCopy.birthDate,
        gender: formDataCopy.gender,
        language: formDataCopy.language,
        cities: formDataCopy.cities,
        fileName: formDataCopy.fileName
      }

      const response = await simulateApiCall(formData)
      submittedForms.value.unshift(formData)
      submitSuccess.value = true
      resetForm()
    } catch (err) {
      submitError.value = err.message || 'Failed to submit form'
    } finally {
      isSubmitting.value = false
    }
  }

  // Form reset
  function resetForm() {
    groups.forEach(group => {
      group.name = ''
      group.email = ''
      group.mobile = ''
    })
    rawDate.value = ''
    gender.value = ''
    language.value = ''
    selectedCities.value = []
    fileName.value = ''
    error.value = ''
    Object.keys(formErrors).forEach(key => {
      if (Array.isArray(formErrors[key])) {
        formErrors[key] = []
      } else {
        formErrors[key] = ''
      }
    })
  }

  // Auth functions
  function simulateLogin() {
    const mockToken = 'mock_jwt_' + Math.random().toString(36).substring(7)
    token.value = mockToken
    localStorage.setItem('jwt_token', mockToken)
  }

  function simulateLogout() {
    token.value = ''
    localStorage.removeItem('jwt_token')
  }
  </script>

  <template>
    <div class="max-w-[1920px] mx-auto p-8">
      <!-- Authentication Status -->
      <div class="mb-8 p-6 bg-gray-100 rounded-lg shadow-sm">
        <div v-if="isAuthenticated" class="flex items-center justify-between">
          <span class="text-green-600 font-medium">Authenticated</span>
          <button 
            @click="simulateLogout" 
            class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 transition-colors"
          >
            Logout
          </button>
        </div>
        <div v-else class="flex items-center justify-between">
          <span class="text-red-600 font-medium">Not Authenticated</span>
          <button 
            @click="simulateLogin" 
            class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600 transition-colors"
          >
            Login
          </button>
        </div>
      </div>

      <!-- Form Container -->
      <form @submit="handleSubmit" class="space-y-8 bg-white p-10 rounded-lg shadow-md">
        <!-- Dynamic form groups -->
        <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
          <InputGroup 
            v-for="(group, index) in groups" 
            :key="group.id" 
            :group="group" 
            :index="index"
            :errors="formErrors.groups[index]"
            @remove="removeGroup(index)" 
          />
        </div>

        <!-- Add more groups button -->
        <div class="flex justify-end">
          <button 
            type="button"
            @click="addGroup" 
            class="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 transition-colors"
          >
            + Add More
          </button>
        </div>

        <!-- Form fields container -->
        <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
          <!-- Birth Date input -->
          <div class="flex flex-col gap-2">
            <label for="birthDate" class="font-medium text-gray-700">Birth Date</label>
            <input 
              id="birthDate" 
              type="date" 
              v-model="rawDate" 
              class="border border-gray-300 px-3 py-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              :class="{ 'border-red-500': formErrors.birthDate }"
            />
            <span v-if="formErrors.birthDate" class="text-red-500 text-sm">{{ formErrors.birthDate }}</span>
          </div>

          <!-- Gender selection -->
          <div class="flex flex-col gap-2">
            <label class="font-medium text-gray-700">Gender</label>
            <div class="flex gap-4">
              <label 
                v-for="option in genderOptions" 
                :key="option" 
                class="flex items-center gap-2 cursor-pointer"
              >
                <input 
                  type="radio" 
                  :value="option" 
                  v-model="gender" 
                  name="gender"
                  class="w-4 h-4 text-blue-600 focus:ring-blue-500"
                />
                <span>{{ option }}</span>
              </label>
            </div>
            <span v-if="formErrors.gender" class="text-red-500 text-sm">{{ formErrors.gender }}</span>
          </div>

          <!-- Language selection -->
          <div class="flex flex-col gap-2">
            <label class="font-medium text-gray-700">Language</label>
            <div class="flex gap-4">
              <label 
                v-for="option in languageOptions" 
                :key="option" 
                class="flex items-center gap-2 cursor-pointer"
              >
                <input 
                  type="radio" 
                  :value="option" 
                  v-model="language" 
                  name="language"
                  class="w-4 h-4 text-blue-600 focus:ring-blue-500"
                />
                <span>{{ option }}</span>
              </label>
            </div>
            <span v-if="formErrors.language" class="text-red-500 text-sm">{{ formErrors.language }}</span>
          </div>

          <!-- Cities multiselect -->
          <div class="flex flex-col gap-2">
            <label class="font-medium text-gray-700">Select Cities</label>
            <Multiselect
              v-model="selectedCities"
              :options="cities"
              :multiple="true"
              :searchable="true"
              :close-on-select="false"
              :clear-on-select="false"
              :preserve-search="true"
              placeholder="Select cities"
              :preselect-first="false"
              :allow-empty="false"
              :max-height="150"
              track-by="value"
              label="label"
              class="multiselect-blue"
            />
            <span v-if="formErrors.cities" class="text-red-500 text-sm">{{ formErrors.cities }}</span>
          </div>
      
          <!-- File upload section -->
          <div class="flex flex-col gap-2 xl:col-span-2">
            <label class="font-medium text-gray-700">Upload File (JPEG or PDF only)</label>
            <div class="flex items-center gap-4">
              <input
                id="file"
                type="file"
                @change="handleFileChange"
                accept=".jpg,.jpeg,.pdf"
                class="border border-gray-300 px-3 py-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                :class="{ 'border-red-500': formErrors.file }"
              />
              <p v-if="fileName" class="text-sm text-gray-600">
                Selected: {{ fileName }}
              </p>
            </div>
            <span v-if="formErrors.file" class="text-red-500 text-sm">{{ formErrors.file }}</span>
            <span v-if="error" class="text-red-500 text-sm">{{ error }}</span>
          </div>
        </div>

        <!-- Form submission status -->
        <div v-if="submitError" class="p-3 bg-red-100 text-red-700 rounded-md">
          {{ submitError }}
        </div>
        <Transition name="fade">
          <div v-if="submitSuccess" class="p-3 bg-green-100 text-green-700 rounded-md">
            Form submitted successfully!
          </div>
        </Transition>

        <!-- Submit button -->
        <div class="flex justify-end">
          <button 
            type="submit"
            :disabled="isSubmitting || !isAuthenticated"
            class="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 transition-colors disabled:bg-gray-400 disabled:cursor-not-allowed relative min-w-[120px]"
          >
            <span v-if="isSubmitting" class="absolute inset-0 flex items-center justify-center">
              <LoadingSpinner />
            </span>
            <span :class="{ 'opacity-0': isSubmitting }">
              {{ isSubmitting ? 'Submitting...' : 'Submit Form' }}
            </span>
          </button>
        </div>
      </form>

      <!-- Submitted Forms Table -->
      <div v-if="submittedForms.length > 0" class="mt-10">
        <h2 class="text-2xl font-semibold text-gray-800 mb-6">Submitted Forms</h2>
        <div class="bg-white rounded-lg shadow-md overflow-hidden">
          <div class="overflow-x-auto">
            <table class="min-w-full divide-y divide-gray-200">
              <thead class="bg-gray-50">
                <tr>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Submission Date
                  </th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Groups
                  </th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Personal Info
                  </th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Preferences
                  </th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    File
                  </th>
                </tr>
              </thead>
              <tbody class="bg-white divide-y divide-gray-200">
                <tr v-for="form in submittedForms" :key="form.id" class="hover:bg-gray-50">
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {{ new Date(form.submittedAt).toLocaleString() }}
                  </td>
                  <td class="px-6 py-4 text-sm text-gray-500">
                    <div v-for="(group, index) in form.groups" :key="index" class="mb-2">
                      <div class="font-medium text-gray-900">{{ group.name }}</div>
                      <div class="text-gray-500">{{ group.email }}</div>
                      <div class="text-gray-500">{{ group.mobile }}</div>
                    </div>
                  </td>
                  <td class="px-6 py-4 text-sm text-gray-500">
                    <div>Birth Date: {{ new Date(form.birthDate).toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' }) }}</div>
                    <div>Gender: {{ form.gender }}</div>
                  </td>
                  <td class="px-6 py-4 text-sm text-gray-500">
                    <div>Language: {{ form.language }}</div>
                    <div>City: {{ form.cities }}</div>
                  </td>
                  <td class="px-6 py-4 text-sm text-gray-500">
                    {{ form.fileName }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </template>

  <style>
  .multiselect-blue {
    --ms-option-bg-selected: #3b82f6;
    --ms-option-color-selected: #ffffff;
    --ms-tag-bg: #3b82f6;
    --ms-tag-color: #ffffff;
    --ms-ring-color: #93c5fd;
    --ms-option-bg-pointed: #f3f4f6;
    --ms-option-color-pointed: #1f2937;
    --ms-option-bg-selected-pointed: #2563eb;
    --ms-option-color-selected-pointed: #ffffff;
    --ms-tag-bg-disabled: #9ca3af;
    --ms-tag-color-disabled: #ffffff;
    --ms-tag-bg-selected: #2563eb;
    --ms-tag-color-selected: #ffffff;
    --ms-tag-bg-selected-pointed: #1d4ed8;
    --ms-tag-color-selected-pointed: #ffffff;
    --ms-tag-bg-selected-disabled: #6b7280;
    --ms-tag-color-selected-disabled: #ffffff;
    --ms-tag-bg-selected-pointed-disabled: #4b5563;
    --ms-tag-color-selected-pointed-disabled: #ffffff;
    --ms-tag-bg-selected-pointed-disabled-selected: #374151;
    --ms-tag-color-selected-pointed-disabled-selected: #ffffff;
    --ms-tag-bg-selected-pointed-disabled-selected-pointed: #1f2937;
    --ms-tag-color-selected-pointed-disabled-selected-pointed: #ffffff;
    --ms-tag-bg-selected-pointed-disabled-selected-pointed-selected: #111827;
    --ms-tag-color-selected-pointed-disabled-selected-pointed-selected: #ffffff;
    --ms-tag-bg-selected-pointed-disabled-selected-pointed-selected-pointed: #0f172a;
    --ms-tag-color-selected-pointed-disabled-selected-pointed-selected-pointed: #ffffff;
  }

  .multiselect-blue .multiselect-dropdown {
    border-color: #e5e7eb;
    border-radius: 0.375rem;
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  }

  .multiselect-blue .multiselect-dropdown:focus-within {
    border-color: #3b82f6;
    box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
  }

  .multiselect-blue .multiselect-option {
    padding: 0.5rem 0.75rem;
    cursor: pointer;
    transition: all 0.2s;
  }

  .multiselect-blue .multiselect-option.is-selected {
    background-color: #3b82f6;
    color: #ffffff;
  }

  .multiselect-blue .multiselect-option.is-pointed {
    background-color: #f3f4f6;
  }

  .multiselect-blue .multiselect-tag {
    background-color: #3b82f6;
    color: #ffffff;
    border-radius: 0.25rem;
    padding: 0.25rem 0.5rem;
    margin: 0.25rem;
    display: inline-flex;
    align-items: center;
    gap: 0.25rem;
  }

  .multiselect-blue .multiselect-tag-icon {
    background-color: rgba(255, 255, 255, 0.2);
    border-radius: 9999px;
    padding: 0.125rem;
    margin-left: 0.25rem;
    cursor: pointer;
    transition: all 0.2s;
  }

  .multiselect-blue .multiselect-tag-icon:hover {
    background-color: rgba(255, 255, 255, 0.3);
  }

  .multiselect-blue .multiselect-search {
    border: none;
    padding: 0.5rem;
    width: 100%;
    outline: none;
    background-color: transparent;
  }

  .multiselect-blue .multiselect-search:focus {
    outline: none;
  }

  .multiselect-blue .multiselect-single-label {
    padding: 0.5rem;
    color: #374151;
  }

  .multiselect-blue .multiselect-multiple-label {
    padding: 0.5rem;
    color: #374151;
  }

  .multiselect-blue .multiselect-placeholder {
    color: #9ca3af;
  }

  .multiselect-blue .multiselect-clear {
    color: #6b7280;
    padding: 0.25rem;
    border-radius: 0.25rem;
    transition: all 0.2s;
  }

  .multiselect-blue .multiselect-clear:hover {
    background-color: #f3f4f6;
    color: #374151;
  }

  .multiselect-blue .multiselect-spinner {
    border-color: #3b82f6;
    border-right-color: transparent;
  }

  .multiselect-blue .multiselect-dropdown-top {
    border-top-left-radius: 0.375rem;
    border-top-right-radius: 0.375rem;
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }

  .multiselect-blue .multiselect-dropdown-bottom {
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    border-bottom-left-radius: 0.375rem;
    border-bottom-right-radius: 0.375rem;
  }

  .overflow-x-auto {
    -webkit-overflow-scrolling: touch;
  }

  table {
    border-collapse: separate;
    border-spacing: 0;
  }

  th {
    position: sticky;
    top: 0;
    z-index: 10;
    background-color: #f9fafb;
  }

  td {
    max-width: 300px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: normal;
  }
  </style>

  <script>
  export function useForm(initialState) {
    const formState = reactive(initialState)
    const errors = reactive({})
    
    const validate = () => {
      // Validation logic
    }
    
    const reset = () => {
      // Reset logic
    }
    
    return {
      formState,
      errors,
      validate,
      reset
    }
  }
  </script>
