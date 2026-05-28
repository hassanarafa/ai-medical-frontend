<template>
  <div class="app-container">
    <ClarinoLanding v-if="!showWizard" @start-wizard="showWizard = true" />

    <div v-else class="wizard-view-wrapper">
      <button class="floating-home-btn" @click="showWizard = false">
        <i class="ph ph-house"></i> Return Home
      </button>

      <DiagnosisWizard />
    </div>
  </div>
</template>

<script>
import ClarinoLanding from "./views/ClarinoLanding.vue";
import DiagnosisWizard from "./views/DiagnosisPage.vue"; // Your original component script 

export default {
  name: "App",
  components: {
    ClarinoLanding,
    DiagnosisWizard
  },
  data() {
    return {
      // Default state renders the landing experience edge-to-edge
      showWizard: false
    };
  }
};
</script>

<style>
/* Global Layout Resets */
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
  background-color: #F4FBF7;
}

.app-container {
  width: 100%;
}

.wizard-view-wrapper {
  position: relative;
  width: 100%;
}

/* Premium Floating Return Action */
.floating-home-btn {
  position: fixed;
  top: 20px;
  left: 20px;
  z-index: 9999;
  background: rgba(45, 90, 67, 0.95);
  color: #ffffff;
  border: none;
  padding: 10px 18px;
  border-radius: 30px;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-weight: 700;
  font-size: 0.85rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 4px 15px rgba(45, 90, 67, 0.2);
  transition: all 0.2s ease;
}

.floating-home-btn:hover {
  background: #76B041;
  transform: translateY(-1px);
}

/* Mobile responsive override for exit button alignment */
@media (max-width: 500px) {
  .floating-home-btn {
    top: auto;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
  }

  .floating-home-btn:hover {
    transform: translateX(-50%) translateY(-2px);
  }
}
</style>