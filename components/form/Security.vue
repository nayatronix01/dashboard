<script>
import RadioGroup from '@/components/form/RadioGroup';
import LabeledInput from '@/components/form/LabeledInput';
import { _VIEW } from '@/config/query-params';
import { mapGetters } from 'vuex';
import LabeledSelect from '@/components/form/LabeledSelect';

export default {
  components: {
    RadioGroup,
    LabeledInput,
    LabeledSelect
  },

  props:      {
    value: {
      type:    Object,
      default: () => {
        return {};
      }
    },
    mode: { type: String, default: 'edit' }
  },

  data() {
    const allCapabilities = ['ALL',
      'AUDIT_CONTROL',
      'AUDIT_WRITE',
      'BLOCK_SUSPEND',
      'CHOWN',
      'DAC_OVERRIDE',
      'DAC_READ_SEARCH',
      'FOWNER',
      'FSETID',
      'IPC_LOCK',
      'IPC_OWNER',
      'KILL',
      'LEASE',
      'LINUX_IMMUTABLE',
      'MAC_ADMIN',
      'MAC_OVERRIDE',
      'MKNOD',
      'NET_ADMIN',
      'NET_BIND_SERVICE',
      'NET_BROADCAST',
      'NET_RAW',
      'SETFCAP',
      'SETGID',
      'SETPCAP',
      'SETUID',
      'SYSLOGSYS_ADMIN',
      'SYS_BOOT',
      'SYS_CHROOT',
      'SYS_MODULE',
      'SYS_NICE',
      'SYS_PACCT',
      'SYS_PTRACE',
      'SYS_RAWIO',
      'SYS_RESOURCE',
      'SYS_TIME',
      'SYS_TTY_CONFIG',
      'WAKE_ALARM'];

    const {
      capabilities = {}, runAsRoot = true, readOnlyRootFilesystem = false, privileged, allowPrivilegeEscalation, runAsUser
    } = this.value;
    const { add = [], drop = [] } = capabilities;

    return {
      privileged, allowPrivilegeEscalation, allCapabilities, runAsRoot, readOnlyRootFilesystem, add, drop, runAsUser
    };
  },

  computed: {
    isView() {
      return this.mode === _VIEW;
    },

    ...mapGetters({ t: 'i18n/t' })
  },

  watch: {
    privileged(neu) {
      if (neu) {
        this.allowPrivilegeEscalation = true;
      }
    }
  },

  methods: {
    update() {
      const securityContext = {
        runAsRoot:                this.runAsRoot,
        readOnlyRootFilesystem:   this.readOnlyRootFilesystem,
        capabilities:             { add: this.add, drop: this.drop },
        privileged:               this.privileged,
        allowPrivilegeEscalation: this.allowPrivilegeEscalation,
        runAsUser:                this.runAsUser
      };

      this.$emit('input', securityContext);
    }
  }
};
</script>

<template>
  <div @input="update">
    <div class="row">
      <div class="col span-6">
        <RadioGroup v-model="privileged" :label="t('workload.container.security.privileged')" :options="[false,true]" :labels="['No', 'Yes: container has full access to the host']" :mode="mode" />
      </div>
      <div class="col span-6">
        <RadioGroup
          v-model="allowPrivilegeEscalation"
          :label="t('workload.container.security.allowPrivilegeEscalation')"
          :disabled="privileged"
          :options="[false,true]"
          :labels="['No', 'Yes: container can gain more privileges than its parent process']"
          :mode="mode"
        />
      </div>
    </div>

    <div class="spacer" />

    <div class="row">
      <div class="col span-6">
        <RadioGroup
          :label="t('workload.container.security.runAsNonRoot')"
          :value="!runAsRoot"
          :options="[false, true]"
          :labels="['No', 'Yes: container must run as a non-root user']"
          :mode="mode"
          @input="e=>runAsRoot = !e"
        />
      </div>
      <div class="col span-6">
        <RadioGroup v-model="readOnlyRootFilesystem" :label="t('workload.container.security.readOnlyRootFilesystem')" :options="[false, true]" :labels="['No', 'Yes: container has a read-only root filesystem']" :mode="mode" />
      </div>
    </div>

    <div class="spacer" />

    <div class="row">
      <div class="col span-6">
        <LabeledInput v-model.number="runAsUser" :label="t('workload.container.security.runAsUser')" :mode="mode" />
      </div>
    </div>

    <div class="spacer" />

    <div class="row mb-0">
      <div class="col span-6">
        <div v-if="isView">
          <label class="text-label"><t k="workload.container.security.addCapabilities" /></label>
          <div v-if="!add.length">
            n/a
          </div>
          <ul v-else class="mt-0 mb-0 pl-15">
            <li v-for="capability in add" :key="capability">
              {{ capability }}
            </li>
          </ul>
        </div>
        <LabeledSelect
          v-else
          v-model="add"
          :multiple="true"
          :label="t('workload.container.security.addCapabilities')"
          :options="allCapabilities"
          :disabled="mode==='view'"
          @input="update"
        />
      </div>
      <div class="col span-6">
        <div v-if="isView">
          <label class="text-label"><t k="workload.container.security.dropCapabilities" /></label>
          <div v-if="!drop.length">
            n/a
          </div>
          <ul v-else class="mt-0 mb-0 pl-15">
            <li v-for="capability in drop" :key="capability">
              {{ capability }}
            </li>
          </ul>
        </div>
        <LabeledSelect
          v-else
          v-model="drop"
          :multiple="true"
          :label="t('workload.container.security.dropCapabilities')"
          :options="allCapabilities"
          :disabled="mode==='view'"
          @input="update"
        />
      </div>
    </div>
  </div>
</template>
