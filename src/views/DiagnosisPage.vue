<template>
  <div class="wizard-container">
    <header class="clinic-header">
      <img :src="clarinoLogo" alt="Clarino" class="header-logo" />
      <div class="header-text">
        <h1>Clarino AI</h1>
        <span>Dermatological Analysis System</span>
      </div>
    </header>

    <div class="tracker-wrapper">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: progressWidth + '%' }"></div>
      </div>
      <div class="steps-count">Step {{ currentStep + 1 }} of {{ steps.length }}</div>
    </div>

    <main class="main-content">
      <transition name="fade-transform" mode="out-in">
        
        <div v-if="!result" class="glass-card" :key="'wizard'">
          <h2 class="step-title">{{ steps[currentStep].title }}</h2>
          
          <div class="step-body">
            <template v-if="currentStep === 0">
              <div class="upload-zone" :class="{ 'has-image': image, 'is-scanning': loading }" @click="!loading && $refs.imageInput.click()">
                <input ref="imageInput" type="file" accept="image/*" hidden @change="onFileChange" />
                
                <div v-if="image" class="preview-container">
                  <img :src="image" class="image-preview" />
                  <div v-if="loading" class="scan-line"></div>
                  <div v-else class="overlay"><i class="ph ph-arrows-clockwise"></i> Tap to Change</div>
                </div>
                
                <div v-else class="upload-prompt">
                  <div class="icon-circle"><i class="ph ph-camera-plus"></i></div>
                  <h3>Capture Skin Area</h3>
                  <p>Ensure good lighting for better AI accuracy</p>
                </div>
              </div>
            </template>

            <template v-else>
              <div class="fields-grid">
                <div v-for="field in currentFields" :key="field.key" class="field-item">
                  <label>{{ field.label }}</label>
                  <div class="select-wrapper">
                    <select v-model="form[field.key]">
                      <option disabled value="">Select an option...</option>
                      <option v-for="opt in field.options" :key="opt">{{ opt }}</option>
                    </select>
                  </div>
                </div>
              </div>
            </template>
          </div>

          <footer class="card-footer">
            <button v-if="currentStep > 0" class="btn-ghost" @click="prevStep" :disabled="loading">Back</button>
            
            <div class="right-actions">
              <button v-if="steps[currentStep].optional && currentStep < lastStep" class="btn-link" @click="skipStep" :disabled="loading">Skip</button>
              
              <button v-if="currentStep < lastStep" class="btn-next" :disabled="currentStep === 0 && !image" @click="nextStep">
                Next <i class="ph ph-caret-right"></i>
              </button>
              
              <button v-if="currentStep === lastStep" class="btn-submit" @click="submitForm" :disabled="loading">
                <span v-if="loading" class="spinner"></span>
                {{ loading ? 'Analyzing Skin...' : 'Start Diagnosis' }}
              </button>
            </div>
          </footer>
        </div>

        <div v-else class="result-dashboard" :key="'result'">
          <div class="report-header">
            <div class="report-id">Ref: #CL-{{ Math.floor(Math.random()*9000) + 1000 }}</div>
            <button class="btn-close" @click="resetWizard">×</button>
          </div>

          <div class="report-grid">
            <section class="diagnosis-section">
              <div class="status-label">Clinical Identification</div>
              <h2 class="condition-name">{{ result.disease }}</h2>
              <p class="reasoning-text">{{ result.description }}</p>
            </section>

            <section class="product-recommendation">
              <div class="product-card-inner">
                <img :src="clarinoLogo" class="product-shot" />
                <div class="suitability-pill" :class="result.suitability.toLowerCase()">
                  {{ result.suitability }}
                </div>
                <p class="suitability-note">Clarino Suitability</p>
              </div>
            </section>
          </div>

          <div class="notes-section">
            <h4>Recommended Care Plan</h4>
            <ul class="care-list">
              <li v-for="(note, i) in result.recommendations" :key="i">
                <i class="ph ph-check-circle"></i> {{ note }}
              </li>
            </ul>
          </div>

          <div class="report-footer">
            <p class="legal">This analysis is AI-generated for guidance and does not replace professional medical consultation.</p>
            <button class="btn-print" @click="window.print()"><i class="ph ph-printer"></i> Print Report</button>
          </div>
        </div>
      </transition>
    </main>
  </div>
</template>

<script>
import axios from "axios";
import clarinoLogo from "../assets/logo.jpeg";

export default {
  data() {
    return {
      clarinoLogo,
      currentStep: 0,
      loading: false,
      result: null,
      image: null,
      rawFile: null,
      steps: [
        { title: "Visual Input", optional: false },
        { title: "Patient Profile", optional: false },
        { title: "Dermal Habits", optional: false },
        { title: "Sensitivity", optional: false },
        { title: "Visual Markers", optional: true },
        { title: "Allergies", optional: true },
        { title: "Medical History", optional: true },
        { title: "Review" }
      ],
      form: { gender: "", age: "", skinType: "", fastFood: "", timeSensitive: "", painful: "", pus: "", redness: "", location: "", allergy: "", longDuration: "", pregnant: "", breastfeeding: "" }
    };
  },
  computed: {
    lastStep() { return this.steps.length - 1; },
    progressWidth() { return ((this.currentStep + 1) / this.steps.length) * 100; },
    currentFields() {
      const map = [[],
      [{ key: "gender", label: "Gender", options: ["Male", "Female"] }, { key: "age", label: "Age", options: ["11-14", "14-18", "18-21", "21-25","25-30" , "30+"] }],
      [{ key: "skinType", label: "Skin Type", options: ["Normal", "Oily", "Dry", "Combination"] }, { key: "fastFood", label: "Eat Fast Food?", options: ["Yes", "No"] }],
      [{ key: "timeSensitive", label: "Acne worse at specific time?", options: ["Yes", "No"] }, { key: "painful", label: "Painful", options: ["Yes", "No"] }],
      [{ key: "pus", label: "Contains pus?", options: ["Yes", "No"] }, { key: "redness", label: "Redness?", options: ["Yes", "No"] }],
      [{ key: "location", label: "Location", options: ["T-Zone", "Forehead", "Cheeks", "Jawline/Chin"] }, { key: "allergy", label: "Allergy?", options: ["None", "Topical Medications", "Fragrances", "Food"] }],
      [{ key: "longDuration", label: "Stays long?", options: ["Yes", "No"] }, { key: "pregnant", label: "Pregnant?", options: ["Yes", "No"] }, { key: "breastfeeding", label: "Breastfeeding?", options: ["Yes", "No"] }]
      ];
      return map[this.currentStep] || [];
    }
  },
  methods: {
    nextStep() { if (this.currentStep < this.lastStep) this.currentStep++; },
    prevStep() { if (this.currentStep > 0) this.currentStep--; },
    skipStep() { this.nextStep(); },
    resetWizard() { this.result = null; this.currentStep = 0; this.image = null; },
    async onFileChange(e) {
      const file = e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = () => (this.image = reader.result);
      reader.readAsDataURL(file);
      this.rawFile = await this.compressImage(file);
    },
    async compressImage(file) {
      return new Promise((resolve) => {
        const img = new Image();
        img.src = URL.createObjectURL(file);
        img.onload = () => {
          const canvas = document.createElement('canvas');
          const MAX = 1200;
          let w = img.width, h = img.height;
          if (w > h) { if (w > MAX) { h *= MAX / w; w = MAX; } } else { if (h > MAX) { w *= MAX / h; h = MAX; } }
          canvas.width = w; canvas.height = h;
          canvas.getContext('2d').drawImage(img, 0, 0, w, h);
          canvas.toBlob(b => resolve(new File([b], file.name, { type: 'image/jpeg' })), 'image/jpeg', 0.8);
        };
      });
    },
    async submitForm() {
      this.loading = true;
      try {
        const fd = new FormData();
        fd.append("image", this.rawFile);
        fd.append("user_answers", JSON.stringify(this.form));
        const res = await axios.post("https://aimedicalrepo-production.up.railway.app/analyze", fd);
        this.result = {
          disease: res.data.diagnosis,
          description: res.data.reasoning,
          suitability: res.data.suitability,
          recommendations: [res.data.clinical_note || "Maintain routine hygiene."],
          medicalAdvice: "This report is AI-assisted. Consult a dermatologist for definitive diagnosis."
        };
      } catch (e) { alert("Analysis failed. Please check connection."); }
      finally { this.loading = false; }
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap');

/* CLARINO BRAND SYSTEM - 2026 
  Primary Dark: #2D5A43 | Primary Leaf: #76B041 | Clinic Mint: #F1F8F4
*/

/* 1. RESET & MASTER CONTAINER */
.wizard-container {
  /* Forces the Mint color to be the absolute floor of the app */
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow-y: auto; /* Enables scrolling for long forms */
  overflow-x: hidden;
  
  margin: 0;
  padding: 40px 20px;
  background-color: #C2E5D3 !important; 
  
  display: flex;
  flex-direction: column;
  align-items: center;
  box-sizing: border-box;
  font-family: 'Plus Jakarta Sans', sans-serif;
  color: #1F2F28;
}

/* 2. THE WATERMARK (Centered Background Branding) */
.wizard-container::after {
  content: "";
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(-15deg);
  
  /* Responsive sizing: 80% of screen width, but capped at 450px */
  width: 80vw;
  height: 80vw;
  max-width: 450px;
  max-height: 450px;
  
  background-image: v-bind("`url(${clarinoLogo})` ");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  
  opacity: 0.04; 
  filter: grayscale(100%);
  pointer-events: none;
  z-index: 0;
}

/* 3. LAYOUT WRAPPERS (Z-index 2 keeps content above watermark) */
.clinic-header, 
.tracker-wrapper, 
.main-content {
  position: relative;
  z-index: 2;
  width: 100%;
  max-width: 580px; /* Limits desktop width for readability */
  box-sizing: border-box;
}

/* 4. CLINIC HEADER */
.clinic-header {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 30px;
}

.header-logo {
  height: 55px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(45, 90, 67, 0.12);
}

.header-text h1 {
  font-size: 1.4rem;
  font-weight: 800;
  margin: 0;
  color: #2D5A43;
}

.header-text span {
  font-size: 0.7rem;
  color: #76B041;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
}

/* 5. TRACKER & PROGRESS BAR */
.tracker-wrapper { margin-bottom: 25px; }
.progress-bar {
  height: 8px;
  background: #E0EADD;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 8px;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #2D5A43, #76B041);
  transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}
.steps-count { font-size: 0.8rem; color: #5C7067; font-weight: 600; }

/* 6. MAIN CARDS (Glassmorphism) */
.glass-card, .result-dashboard {
  width: 100%;
  background: rgba(255, 255, 255, 0.96);
  backdrop-filter: blur(10px);
  border-radius: 28px;
  padding: 40px;
  border: 1px solid #E9F0E8;
  box-shadow: 0 20px 50px rgba(45, 90, 67, 0.05);
  box-sizing: border-box;
}

.step-title {
  font-size: 1.5rem;
  font-weight: 800;
  margin-bottom: 30px;
  color: #2D5A43;
}

/* 7. UPLOAD ZONE & PREVIEW */
.upload-zone {
  width: 100%;
  border: 2px dashed #B5C9BE;
  border-radius: 20px;
  padding: 30px 15px;
  text-align: center;
  cursor: pointer;
  background: #F1F8F4;
  transition: all 0.3s ease;
  box-sizing: border-box;
}
.upload-zone:hover { border-color: #76B041; background: #E8F5E9; }

.preview-container { border-radius: 16px; overflow: hidden; position: relative; }
.image-preview { width: 100%; max-height: 350px; object-fit: cover; display: block; }
.overlay {
  position: absolute; bottom: 0; width: 100%; background: rgba(31, 47, 40, 0.7);
  color: white; padding: 10px; font-size: 0.8rem; font-weight: 600; text-align: center;
}

/* 8. FORM FIELDS */
.fields-grid { display: grid; gap: 20px; }
.field-item label { display: block; font-size: 0.85rem; font-weight: 700; color: #2D5A43; margin-bottom: 8px; }
.select-wrapper select {
  width: 100%; padding: 14px; border-radius: 12px; border: 1px solid #D1DBD5;
  background: white; font-size: 0.95rem; appearance: none;
  box-sizing: border-box;
}

/* 9. BUTTONS & ACTIONS */
.card-footer { 
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
  margin-top: 40px; 
}
.btn-next, .btn-submit {
  padding: 14px 30px; border-radius: 12px; background: #2D5A43; color: white;
  border: none; font-weight: 700; cursor: pointer; transition: 0.2s;
}
.btn-next:hover, .btn-submit:hover { background: #76B041; transform: translateY(-2px); }
.btn-ghost { background: none; border: none; color: #2D5A43; font-weight: 700; cursor: pointer; }
.btn-link { color: #5C7067; font-size: 0.85rem; font-weight: 600; text-decoration: underline; cursor: pointer; background: none; border: none; }

.right-actions { display: flex; align-items: center; gap: 15px; }

/* 10. RESULT DASHBOARD SPECIFICS */
.condition-name { 
  font-size: 1.8rem; font-weight: 800; color: #2D5A43; margin: 10px 0;
  border-left: 5px solid #76B041; padding-left: 15px;
}
.product-card-inner { background: #F1F8F4; padding: 20px; border-radius: 20px; text-align: center; }
.product-shot { width: 100%; max-width: 140px; border-radius: 10px; margin-bottom: 10px; }
.suitability-pill { padding: 6px 15px; border-radius: 20px; font-size: 0.7rem; font-weight: 800; text-transform: uppercase; }
.suitability-pill.safe { background: #DCF2E1; color: #1B4332; }
.suitability-pill.unsafe { background: #FEE2E2; color: #991B1B; }

/* 📱 NARROW SCREEN (MOBILE) HANDLING */
@media (max-width: 500px) {
  .wizard-container { padding: 20px 10px; } /* Tighten edge spacing */
  
  .clinic-header { 
    flex-direction: column; 
    text-align: center; 
    margin-bottom: 20px;
    gap: 10px;
  }
  
  .glass-card, .result-dashboard { 
    padding: 25px 15px; 
    border-radius: 20px; 
  }
  
  .step-title { font-size: 1.2rem; text-align: center; }
  
  .card-footer { 
    flex-direction: column-reverse; /* Stack "Back" under the main buttons */
    gap: 20px; 
  }
  
  .right-actions { 
    width: 100%; 
    flex-direction: column; 
    gap: 15px;
  }
  
  .btn-next, .btn-submit { width: 100%; padding: 16px; }
  .btn-link { text-align: center; width: 100%; }

  .report-grid { grid-template-columns: 1fr; gap: 20px; }
  .product-recommendation { order: -1; }
  .condition-name { font-size: 1.5rem; }
}

/* 🧬 UTILITIES & ANIMATIONS */
.spinner {
  width: 18px; height: 18px; border: 3px solid rgba(255,255,255,0.3);
  border-top-color: #fff; border-radius: 50%; display: inline-block;
  animation: spin 0.8s linear infinite; margin-right: 8px;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* TRANSITIONS */
.fade-transform-enter-active, .fade-transform-leave-active { transition: all 0.3s ease; }
.fade-transform-enter-from { opacity: 0; transform: translateY(15px); }
.fade-transform-leave-to { opacity: 0; transform: translateY(-15px); }
</style>