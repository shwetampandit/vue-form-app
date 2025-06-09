<template>
  <div class="bg-gray-50 p-4 rounded-lg shadow-sm">
    <!-- Group header with remove button -->
    <div class="flex justify-between items-center mb-4">
      <h3 class="text-lg font-medium text-gray-900">Group {{ index + 1 }}</h3>
      <button 
        type="button"
        @click="$emit('remove')"
        class="text-red-600 hover:text-red-800 transition-colors"
      >
        Remove
      </button>
    </div>

    <!-- Form fields -->
    <div class="space-y-4">
      <!-- Name input -->
      <div class="flex flex-col gap-1">
        <label :for="'name-' + index" class="font-medium text-gray-700">Name</label>
        <input 
          :id="'name-' + index"
          type="text"
          :value="group.name"
          @input="$emit('update:group', { ...group, name: $event.target.value })"
          class="border border-gray-300 px-3 py-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          :class="{ 'border-red-500': errors.name }"
        />
        <span v-if="errors.name" class="text-red-500 text-sm">{{ errors.name }}</span>
      </div>

      <!-- Email input -->
      <div class="flex flex-col gap-1">
        <label :for="'email-' + index" class="font-medium text-gray-700">Email</label>
        <input 
          :id="'email-' + index"
          type="email"
          :value="group.email"
          @input="$emit('update:group', { ...group, email: $event.target.value })"
          class="border border-gray-300 px-3 py-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          :class="{ 'border-red-500': errors.email }"
        />
        <span v-if="errors.email" class="text-red-500 text-sm">{{ errors.email }}</span>
      </div>

      <!-- Mobile input -->
      <div class="flex flex-col gap-1">
        <label :for="'mobile-' + index" class="font-medium text-gray-700">Mobile</label>
        <input 
          :id="'mobile-' + index"
          type="tel"
          :value="group.mobile"
          @input="$emit('update:group', { ...group, mobile: $event.target.value })"
          class="border border-gray-300 px-3 py-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          :class="{ 'border-red-500': errors.mobile }"
        />
        <span v-if="errors.mobile" class="text-red-500 text-sm">{{ errors.mobile }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
// Props for form group data and validation
defineProps({
  group: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  errors: {
    type: Object,
    default: () => ({})
  }
})

// Emit events for group management
const emit = defineEmits(['update:group', 'remove'])
</script>

<style scoped>
/* Input focus styles */
input:focus {
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}

/* Error state styles */
input.border-red-500:focus {
  box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.2);
}
</style>