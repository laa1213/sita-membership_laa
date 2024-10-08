<script setup>
import { ref, computed } from 'vue'
import { useForm, Link, usePage } from '@inertiajs/vue3'
import { FwbAlert, FwbListGroup, FwbListGroupItem, FwbButton, FwbTab, FwbTabs } from 'flowbite-vue'
import MemberSummaryCard from '@/Components/MemberSummaryCard.vue'
import CheckCircleOutlineIcon from 'vue-material-design-icons/CheckCircleOutline.vue'
import AlertCircleOutlineIcon from 'vue-material-design-icons/AlertCircleOutline.vue'
import AcceptModal from '@/Components/AcceptModal.vue'
import RejectionModal from '@/Components/RejectionModal.vue'
import CardBox from '@/Components/CardBox.vue'
import MemberPayment from '@/Components/MemberPayment.vue'
import MemberInvoice from '@/Components/MemberInvoice.vue'
import SetMembershipStatus from '@/Components/SetMembershipStatus.vue'

const props = defineProps(['member', 'options', 'data', 'statuses', 'membershipStatuses', 'invoices'])

const completion = props.options.completion.data
const showAcceptanceModal = ref(false)
const showActivateModal = ref(false)
const showRejectionModal = ref(false)
const permissions = usePage().props.user.permissions

const form = useForm({})

function submit() {
  form.put(route('members.submit', props.member.id), {
    resetOnSuccess: false,
  })
}
function endorse() {
  form.put(route('members.endorse', props.member.id), {
    resetOnSuccess: false,
  })
}
function sendSubReminder() {
  form.put(route('members.send-sub-reminder', props.member.id), {
    resetOnSuccess: false,
  })
}
function sendPastDueSubReminder() {
  form.put(route('members.send-past-due-sub-reminder', props.member.id), {
    resetOnSuccess: false,
  })
}
function convertToDraft() {
  form.put(route('members.convert-to-draft', props.member.id), {
    resetOnSuccess: false,
  })
}

const application_status_id = computed(() => {
  let m = props.member
  if (m.membership_status_id) {
    return m.membership_status_id
  }
  return 0
})

const application_ready_for_submission = props.options.completion.overall.status

const activeTab = ref('first')
</script>

<template>
  <CardBox>
    <!-- Alerts -->
    <fwb-alert v-if="application_status_id === 0" type="info" class="mb-6">
      <strong>Draft</strong><br />
      Complete your signup and press Submit.
    </fwb-alert>

    <fwb-alert v-if="application_status_id === 2" type="info" class="mb-6">
      <strong>Submitted</strong><br />
      Your application has been submitted and is under review.
    </fwb-alert>

    <fwb-alert v-if="application_status_id === 3" type="info" class="mb-6">
      <strong>Endorsed</strong><br />
      Your application has been endorsed and is awaiting settlement.
    </fwb-alert>

    <fwb-alert v-if="application_status_id === 8 || props.data.reason != null" type="danger" class="mb-6">
      <strong>Rejected</strong><br />
      Your application was rejected. See below for more information:
      <ul v-if="props.data.reason != null" class="list-group">
        <li class="list-item mt-2">"{{ props.data.reason }}"</li>
      </ul>
      <br />
      <p v-if="application_status_id === 8" class="underline bold cursor-pointer" @click="convertToDraft">I understand and I would like to proceed to update my membership application</p>
    </fwb-alert>

    <MemberSummaryCard :member="props.member" link-route="members.signup.index" class="mb-6" />

    <fwb-tabs v-model="activeTab" class="p-5">
      <fwb-tab name="first" title="General">
        <!-- General details -->
        <fwb-list-group class="w-full">
          <Link v-for="(c, index) in completion" :href="route('members.signup.index', { member: props.member.id, tab: index.replace('part', '') })">
            <fwb-list-group-item>
              <template #prefix>
                <CheckCircleOutlineIcon v-if="c.status" fill-color="green" />
                <AlertCircleOutlineIcon v-else fill-color="blue" />
              </template>
              {{ c.title }}
            </fwb-list-group-item>
          </Link>
        </fwb-list-group>
      </fwb-tab>
      <fwb-tab name="fourth" title="Membership Status" v-if="permissions.canUpdateMembershipStatus">
        <!-- SetMembershipStatus -->
        <SetMembershipStatus :membershipStatuses="props.membershipStatuses" :status="props.member.membership_status_id" :member_id="props.member.id" />
      </fwb-tab>
      <fwb-tab name="second" title="Billing">
        <!-- MemberInvoice -->
        <MemberInvoice :invoices="props.invoices" :member_id="props.member.id" />
        <!-- MemberPayment -->
        <MemberPayment :statuses="props.statuses" />
      </fwb-tab>
      <fwb-tab name="third" title="Audit">
        <!-- Audit log link -->
        <Link :href="route('members.audit.index', { member: member.id })" class="underline text-indigo-500 text-sm mt-5">View audit log</Link>
      </fwb-tab>
    </fwb-tabs>

    <template #footer>
      <p v-show="!application_ready_for_submission" class="w-full mb-6 ml-2 text-sm text-gray-500">Please ensure all sections are completed before submitting</p>

      <!-- Action buttons -->
      <fwb-button v-if="application_status_id <= 1" class="w-full mb-6" :disabled="!(application_ready_for_submission || $page.props.user.permissions.canSubmit) || form.processing" default @click.prevent="submit">Submit</fwb-button>
      <fwb-button v-if="application_status_id === 2 && $page.props.user.permissions.canEndorse" class="w-full mb-3" :disabled="form.processing" default @click.prevent="endorse">Endorse</fwb-button>
      <fwb-button v-if="application_status_id === 2 && $page.props.user.permissions.canReject" class="w-full mb-6 bg-red-500 hover:bg-red-400" :disabled="form.processing" default @click.prevent="showRejectionModal = true">Reject</fwb-button>
      <fwb-button v-if="application_status_id === 3 && $page.props.user.permissions.canAccept" class="w-full mb-6" :disabled="form.processing" default @click.prevent="showAcceptanceModal = true">Accept</fwb-button>
      <fwb-button v-if="(application_status_id === 5 || application_status_id === 6) && $page.props.user.permissions.canAccept" class="w-full mb-6" color="green" :disabled="form.processing" default @click.prevent="showActivateModal = true">Activate Membership</fwb-button>
      <fwb-button v-if="application_status_id === 4 && $page.props.user.permissions.canSendSubReminder" class="w-full mb-6" :disabled="form.processing" default @click.prevent="sendSubReminder">Send sub reminder</fwb-button>
      <fwb-button v-if="application_status_id === 5 && $page.props.user.permissions.canSendPastDueSubReminder" class="w-full mb-6" :disabled="form.processing" default @click.prevent="sendPastDueSubReminder">Send past due sub reminder</fwb-button>

      <!-- Dialog Modals -->
      <AcceptModal :show="showAcceptanceModal" :member-id="member.id" @close="showAcceptanceModal = false" />
      <AcceptModal :show="showActivateModal" :member-id="member.id" heading-text="Activate Membership" button-text="Activate" @close="showActivateModal = false" />
      <RejectionModal :show="showRejectionModal" :member-id="member.id" @close="showRejectionModal = false" />
    </template>
  </CardBox>
</template>
