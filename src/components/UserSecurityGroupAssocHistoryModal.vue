<template>
  <ion-header>
    <ion-toolbar>
      <ion-buttons slot="start">
        <ion-button @click="closeModal"> 
          <ion-icon slot="icon-only" :icon="closeOutline" />
        </ion-button>
      </ion-buttons>
      <ion-title>{{ translate("Security group history") }}</ion-title>
    </ion-toolbar>
  </ion-header>

  <ion-content>
    <ion-list v-if="userGroupAssocHistories.length">
      <ion-item v-for="assocHistory in userGroupAssocHistories" :key="assocHistory.groupId">
        <ion-label>
          {{ assocHistory.groupName ? assocHistory.groupName : assocHistory.groupId }}
          <p>{{ assocHistory.groupId }}</p>
        </ion-label>
        <ion-note slot="end">{{ getDateWithOrdinalSuffix(assocHistory.fromDate) }} - {{ assocHistory.thruDate ? getDateWithOrdinalSuffix(assocHistory.thruDate) : translate('Current') }}</ion-note>
      </ion-item>
    </ion-list>
    <div class="empty-state" v-else>
      <p>{{ translate("No history found.") }}</p>
    </div>
  </ion-content>
</template>

<script lang="ts">
import { 
  IonButtons,
  IonButton,
  IonContent,
  IonHeader,
  IonIcon,
  IonItem,
  IonLabel,
  IonList,
  IonNote,
  IonTitle,
  IonToolbar,
  modalController
} from "@ionic/vue";
import { defineComponent } from "vue";
import {
  closeOutline,
  eyeOutline,
  eyeOffOutline,
  lockClosedOutline,
  mailOutline
} from "ionicons/icons";
import { mapGetters, useStore } from "vuex";
import { translate } from '@hotwax/dxp-components'
import { getDateWithOrdinalSuffix } from "@/utils";
import { hasError } from "@/adapter";
import { UserService } from "@/services/UserService";
import { Actions, hasPermission } from '@/authorization'
export default defineComponent({
  name: "UserSecurityGroupAssocHistoryModal",
  components: { 
    IonButtons,
    IonButton,
    IonContent,
    IonHeader,
    IonIcon,
    IonItem,
    IonLabel,
    IonList,
    IonNote,
    IonTitle,
    IonToolbar,
  },
  computed: {
    ...mapGetters({
      selectedUser: 'user/getSelectedUser',
      securityGroups: 'util/getSecurityGroups',
    })
  },
  data() {
    return {
      userGroupAssocHistories: [] as any
    }
  },
  mounted() {
    this.fetchUserSecurityGroupAssoHistory()
  },
  methods: {
    closeModal() {
      modalController.dismiss({ dismissed: true});
    },
    async fetchUserSecurityGroupAssoHistory() {
      if(!this.selectedUser.userLoginId) return;

      const securityGroupNameByGroupId = {} as any;
      let userGroupAssocHistories = [] as any;
      try {
        const resp = await UserService.fetchUserSecurityGroupAssocHistory({
          entityName: "UserLoginAndSecurityGroup",
          inputFields: {
            userLoginId: this.selectedUser.userLoginId,
          },
          orderBy: "thruDate DESC",
          viewSize: 250
        })
        if(!hasError(resp)) {
          userGroupAssocHistories = resp.data.docs
          this.securityGroups.map((group: any) => securityGroupNameByGroupId[group.groupId] = group.groupName);
          userGroupAssocHistories.map((history: any) => {
            history["groupName"] = securityGroupNameByGroupId[history.groupId]
          })
          const currentSecurityGroups = userGroupAssocHistories.filter((history: any) => !history.thruDate);
          const expiredSecurityGroups = userGroupAssocHistories.filter((history: any) => history.thruDate);
          userGroupAssocHistories = currentSecurityGroups.concat(expiredSecurityGroups);
        } else {
          throw resp.data;
        }
      } catch(error: any) {
        console.error(error);
      }
      this.userGroupAssocHistories = userGroupAssocHistories
    }
  },
  setup() {
    const store = useStore();
    return {
      closeOutline,
      eyeOutline,
      eyeOffOutline,
      getDateWithOrdinalSuffix,
      hasPermission,
      lockClosedOutline,
      mailOutline,
      store,
      translate,
      Actions
    };
  },
});
</script>