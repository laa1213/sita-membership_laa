<script setup>
import { useForm, Head, Link } from '@inertiajs/vue3'
import { mdiEmail } from '@mdi/js'
import LayoutGuest from '@/Layouts/LayoutGuest.vue'
import SectionFullScreen from '@/Components/SectionFullScreen.vue'
import CardBox from '@/Components/CardBox.vue'
import FormField from '@/Components/FormField.vue'
import FormControl from '@/Components/FormControl.vue'
import BaseDivider from '@/Components/BaseDivider.vue'
import BaseButton from '@/Components/BaseButton.vue'
import FormValidationErrors from '@/Components/FormValidationErrors.vue'
import NotificationBarInCard from '@/Components/NotificationBarInCard.vue'
import BaseLevel from '@/Components/BaseLevel.vue'
import AuthenticationCardLogo from '@/Components/AuthenticationCardLogo.vue'

defineProps({
  status: {
    type: String,
    default: null,
  },
})

const form = useForm({
  email: '',
})

const submit = () => {
  form.post(route('password.email'))
}
</script>

<template>
  <LayoutGuest>
    <Head title="Forgot Password" />

    <SectionFullScreen v-slot="{ cardClass }" bg="purplePink">
      <CardBox :class="cardClass" is-form @submit.prevent="submit">
        <FormValidationErrors />

        <NotificationBarInCard v-if="status" color="info">
          {{ status }}
        </NotificationBarInCard>

        <AuthenticationCardLogo />

        <FormField>
          <div class="mb-4 text-sm text-gray-600">Forgot your password? No problem. Just let us know your email address and we will email you a password reset link that will allow you to choose a new one.</div>
        </FormField>

        <FormField label="Email" help="Please enter your email">
          <FormControl v-model="form.email" :icon="mdiEmail" autocomplete="email" type="email" required />
        </FormField>

        <BaseDivider />

        <BaseLevel>
          <BaseButton type="submit" color="info" label="Send link" :class="{ 'opacity-25': form.processing }" :disabled="form.processing" />
          <Link :href="route('login')" class="text-sm"> Back to login </Link>
        </BaseLevel>
      </CardBox>
    </SectionFullScreen>
  </LayoutGuest>
</template>
