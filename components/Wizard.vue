<script>
import { STEP } from '@/config/query-params';
/*
Wizard accepts an array of steps (see props), and creates named slots for each step.
It also contains slots for buttons:
    next
    cancel
    finish

Wizard will emit these events:
    next({step})
    cancel
    finish
*/

export default {
  props:      {
    mode: {
      type:    String,
      default: 'create'
    },

    /*
  steps need: {
    name: String - this will be the slot name
    label: String - this will appear in the top nav bar below circles
    subtext: String (optional) - If defined, appears below the step number in the banner. If blank, label is used
    ready: Boolean - whether or not the step is completed/wizard is able to go to next step
      if a step has ready=true, the wizard also allows navigation *back* to it
  }
  */
    steps: {
      type:    Array,
      default: () => []
    },

    // Initial step to show when Wizard loads. This is overruled by the 'step' query param, if available
    initStepIdx: {
      type:    Number,
      default: 0
    },

    // if true, allow users to navigate back to the first step of the Wizard
    // if false, only way back to step 1 is to cancel and undo all configuration
    editFirstStep: {
      type:    Boolean,
      default: false
    },

    // place the same title (e.g. the type of thing being created by wizard) on every page
    bannerTitle: {
      type:    String,
      default: null
    },

    // circular image left of banner title
    bannerImage: {
      type:    String,
      default: null
    }
  },

  data() {
    const queryStep = this.$route.query[STEP];

    const activeStep = this.steps[queryStep - 1] || this.steps[this.initStepIdx];

    return { activeStep };
  },

  computed: {
    activeStepIndex() {
      return this.steps.indexOf(this.activeStep);
    },

    canNext() {
      return (this.activeStepIndex < this.steps.length - 1) && this.activeStep.ready;
    },

    readySteps() {
      return this.steps.filter(step => step.ready);
    }

  },

  watch: {
    '$route.query'(neu = {}) {
      this.goToStep(neu[STEP]);
    },
  },

  methods: {
    goToStep(number, fromNav) {
      if (number < 1) {
        return;
      }

      // if editFirstStep is false, do not allow returning to step 1 (restarting wizard) from top nav
      if (!this.editFirstStep && (number === 1 && fromNav)) {
        return;
      }

      const {
        steps,
        $route: { query:{ step: queryStep } },
      } = this;

      const selected = steps[number - 1];

      if ( !selected || !this.isAvailable(selected)) {
        return;
      }

      if (queryStep !== number) {
        this.$router.applyQuery({ [STEP]: number });
      }

      this.activeStep = selected;

      this.$emit('next', { step: selected });
    },

    cancel() {
      this.goToStep(1);
      this.$emit('cancel');
    },

    finish() {
      this.$emit('finish');
    },

    next() {
      this.goToStep(this.activeStepIndex + 2);
    },

    back() {
      this.goToStep(this.activeStepIndex);
    },

    // a step is not available if ready=false for any previous steps OR if the editFirstStep=false and it is the first step
    isAvailable(step) {
      if (!step) {
        return false;
      }

      const idx = this.steps.indexOf(step);

      if (idx === 0 && !this.editFirstStep) {
        return false;
      }

      for (let i = 0; i < idx; i++) {
        if (!this.steps[i].ready) {
          return false;
        }
      }

      return true;
    },
  }
};
</script>

<template>
  <div>
    <ul
      class="steps"
      tabindex="0"
      @keyup.right.stop="selectNext(1)"
      @keyup.left.stop="selectNext(-1)"
    >
      <template v-for="(step, idx ) in steps">
        <li

          :id="step.name"
          :key="step.name+'li'"
          :class="{step: true, active: step === activeStep, disabled: !isAvailable(step)}"
          role="presentation"
        >
          <span
            :aria-controls="'step' + idx+1"
            :aria-selected="step === activeStep"
            role="tab"
            class="controls"
            @click.prevent="goToStep(idx+1, true)"
          >
            <span class="icon icon-lg" :class="{'icon-dot': step !== activeStep, 'icon-dot-open':step === activeStep}" />
            <span>
              {{ step.label }}
            </span>
          </span>
        </li>
        <div v-if="idx!==steps.length-1" :key="step.name" class="divider" />
      </template>
    </ul>

    <div class="spacer" />

    <div class="top choice-banner">
      <div v-if="bannerTitle || bannerImage" class="title">
        <div v-if="bannerImage" class="round-image">
          <img :src="bannerImage" />
        </div>
        <h2 v-if="bannerTitle">
          {{ bannerTitle }}
        </h2>
      </div>
      <div class="subtitle">
        <h2> {{ t('wizard.step', {number:activeStepIndex+1}) }}</h2>
        <span class="subtext">{{ activeStep.subtext || activeStep.label }}</span>
      </div>
    </div>

    <div class="spacer" />

    <div class="step-container">
      <div v-for="step in steps" :key="step.name">
        <template v-if="step === activeStep">
          <slot :step="step" :name="step.name" />
        </template>
      </div>
    </div>

    <div class="spacer" />

    <div class="controls-row">
      <slot name="cancel" :cancel="cancel">
        <button v-if="activeStepIndex" type="button" class="btn role-secondary" @click="cancel">
          <t k="generic.cancel" />
        </button>
      </slot>

      <div>
        <slot v-if="activeStepIndex!==0" name="back" :back="back">
          <button :disabled="!editFirstStep && activeStepIndex===1" type="button" class="btn role-secondary" @click="back()">
            <t k="wizard.back" />
          </button>
        </slot>
        <slot v-if="activeStepIndex === steps.length-1" name="finish" :finish="finish">
          <button :disabled="!activeStep.ready" type="button" class="btn role-primary" @click="finish">
            <t k="wizard.finish" />
          </button>
        </slot>
        <slot v-else name="next" :next="next">
          <button :disabled="!canNext" type="button" class="btn role-primary" @click="next()">
            <t k="wizard.next" />
          </button>
        </slot>
      </div>
    </div>
  </div>
</template>

<style lang='scss'>

.choice-banner {
  padding: 10px;
  margin: 10px;
  flex-basis: 20%;
  border-radius: var(--border-radius);
  border-left: 5px solid var(--primary);
  display: flex;

  &.top {
    background-image: linear-gradient(-90deg, var(--body-bg) , var(--accent-btn));

    H2 {
      margin: 0px;
    }

    .title{
      flex-basis: 10%;
      border-right: 1px solid var(--primary);
      margin-right: 20px;
      padding-right: 20px;
      display: flex;
      align-items: center;
      justify-content: space-evenly;
    }

    .subtitle{
      display: flex;
      flex-direction: column;
      justify-content: center;
      & .subtext {
        color: var(--input-label);
      }
    }

  }

  &:not(.top){
    box-shadow: 0px 0px 12px 3px var(--box-bg);
    flex-direction: row;
    align-items: center;
    justify-content: start;
  }

  & .round-image {
    width: 50px;
    height: 50px;
    margin: 10px;
    border-radius: 50%;
    overflow: hidden;
  }
}

.steps {
    margin: 0 20% 0 20%;;
    display:flex;
    justify-content: space-between;
    list-style-type:none;
    padding: 0;

    &:focus{
        outline:none;
        box-shadow: none;
    }

    & li.step{
      display: flex;
      flex-direction: row;
      flex-grow: 1;
      align-items: center;

      &:last-of-type{
        flex-grow: 0;
      }

      & .controls {
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 40px;
        overflow: visible;
        & > span {
          padding-bottom: 10px;
          white-space: nowrap;
        }
      }

      &.active .controls{
        color: var(--primary);
      }

      &:not(.disabled){
        & .controls:hover>*{
            color: var(--primary) !important;
            cursor: pointer;
        }
      }

      &:not(.active) {
        & .controls>*{
          color: var(--input-disabled-text);
          text-decoration: none;
        }
      }
    }

    & .divider {
      flex-basis: 100%;
      border-top: 1px solid var(--border);
      position: relative;
      top: 5px;
    }
}

.controls-row {
  display: flex;
  justify-content: space-between;
}
</style>
