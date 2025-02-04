<template>
  <div :id="`user${member.memberID}`" class="component" :style="{ width: width }">
    <Component :is="child" :member="member" :props="props" />
    <b-popover
      v-if="showPopover"
      :target="`user${member.memberID}`"
      triggers="hover"
      placement="top"
      boundary="viewport"
    >
      <b-button class="rounded-circle px-2" variant="danger" @click="openModal">
        <b-icon icon="x" scale="2" />
      </b-button>
    </b-popover>
    <b-modal v-if="showModal" :id="`modal${member.memberID}`">
      <template #modal-header>
        <h4>{{ t("page.session.during.modal.title") }}</h4>
      </template>
      {{ t("page.session.during.modal.bodyPart1") }}
      <b>{{ member.name }}</b>
      {{ t("page.session.during.modal.bodyPart2") }}
      <template #modal-footer>
        <b-button @click="closeModal">
          {{ t("page.session.during.modal.buttons.cancel") }}
        </b-button>
        <b-button variant="danger" @click="removeMember">
          {{ t("page.session.during.modal.buttons.ok") }}
        </b-button>
      </template>
    </b-modal>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";
import Constants from "@/constants";
import { useDiveniStore } from "@/store";
import Member from "@/model/Member";
import { useI18n } from "vue-i18n";

export default defineComponent({
  components: {
    RoundedAvatar: () => import("@/components/RoundedAvatar.vue"),
    SessionMemberCard: () => import("@/components/SessionMemberCard.vue"),
  },
  props: {
    child: { type: String, required: true },
    member: { type: Object as PropType<Member>, required: true },
    props: { type: Object, required: false, default: () => ({}) },
  },
  setup() {
    const store = useDiveniStore();
    const { t } = useI18n();
    return { store, t };
  },
  data() {
    return {
      showModal: false,
      showPopover: true,
    };
  },
  computed: {
    width() {
      switch (this.child) {
        case "RoundedAvatar":
          return "100px";
        case "SessionMemberCard":
          return "190px";
        default:
          return "100px";
      }
    },
  },
  methods: {
    openModal() {
      this.showModal = true;
      this.showPopover = false;
    },
    closeModal() {
      this.showModal = false;
    },
    removeMember() {
      const endPoint = Constants.webSocketKickMemberRoute;
      this.store.sendViaBackendWS(endPoint, this.member.memberID);
      this.closeModal();
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS/SCSS to this component only -->
<style lang="scss" scoped>
.popover {
  background: transparent;
  border: transparent;
}
.component {
  display: grid;
  justify-content: center;
  align-items: center;
}
</style>
