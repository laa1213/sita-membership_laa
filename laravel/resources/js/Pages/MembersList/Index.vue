<script setup>
import MembersList from '@/Components/MembersList.vue'
import { mdiViewList } from '@mdi/js'
import SectionMain from '@/Components/SectionMain.vue'
import SectionTitleLineWithButton from '@/Components/SectionTitleLineWithButton.vue'
import LayoutGuestAlt from '@/Layouts/LayoutGuestAlt.vue'
import { useStyleStore } from '@/Stores/style.js'

defineProps({
  members: Object,
  canLogin: Boolean,
  canRegister: Boolean,
  appVersion: String,
})

const styleStore = useStyleStore()
</script>

<template>
  <LayoutGuestAlt title="Members" :canLogin="true" :canRegister="true">
    <SectionMain>
      <SectionTitleLineWithButton :icon="mdiViewList" title="SITA Members" main> &nbsp; </SectionTitleLineWithButton>
      <MembersList :list="members" searchRoute="members-list.index" />
    </SectionMain>
  </LayoutGuestAlt>
</template>
