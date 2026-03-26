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
              <div class="upload-zone" :class="{ 'has-image': image }" @click="$refs.imageInput.click()">
                <input ref="imageInput" type="file" accept="image/*" hidden @change="onFileChange" />

                <div v-if="image" class="preview-container">
                  <img :src="image" class="image-preview" />
                  <div class="overlay"><i class="ph ph-arrows-clockwise"></i> Tap to Change</div>
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
            <button v-if="currentStep > 0" class="btn-ghost" @click="prevStep">Back</button>

            <div class="right-actions">
              <button v-if="steps[currentStep].optional && currentStep < lastStep" class="btn-link" @click="skipStep">
                Skip this step
              </button>
              <button v-if="currentStep < lastStep" class="btn-next" :disabled="currentStep === 0 && !image"
                @click="nextStep">
                Next <i class="ph ph-caret-right"></i>
              </button>

              <button v-if="currentStep === lastStep" class="btn-submit" @click="submitForm" :disabled="loading">
                <span v-if="loading" class="spinner"></span>
                {{ loading ? 'Analyzing...' : 'Start Diagnosis' }}
              </button>
            </div>
          </footer>
        </div>

        <div v-else class="result-dashboard" :key="'result'">
          <!-- <div class="report-header">
            <div class="report-id">Ref: #AI-{{ Math.floor(Math.random()*10000) }}</div>
            <button class="btn-close" @click="resetWizard">×</button>
          </div> -->

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
                <p class="suitability-note">Clarino Treatment Recommendation</p>
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
            <p class="legal">This analysis is AI-generated for educational guidance and does not replace professional
              medical
              consultation.</p>
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
      [{ key: "gender", label: "Gender", options: ["Male", "Female"] }, { key: "age", label: "Age Group", options: ["11-14", "14-18", "18-21", "21-30", "30+"] }],
      [{ key: "skinType", label: "Baseline Skin Type", options: ["Normal", "Oily", "Dry", "Combination"] }, { key: "fastFood", label: "Frequent High-Sugar/Fat Diet?", options: ["Yes", "No"] }],
      [{ key: "timeSensitive", label: "Hormonal/Periodic Cycles?", options: ["Yes", "No"] }, { key: "painful", label: "Inflammatory Pain?", options: ["Yes", "No"] }],
      [{ key: "pus", label: "Pustule Formation?", options: ["Yes", "No"] }, { key: "redness", label: "Persistent Erythema (Redness)?", options: ["Yes", "No"] }],
      [{ key: "location", label: "Primary Cluster", options: ["T-Zone", "Forehead", "Cheeks", "Jawline/Chin"] }, { key: "allergy", label: "Known Hypersensitivities?", options: ["None", "Topical Medications", "Fragrances", "Food"] }],
      [{ key: "longDuration", label: "Chronic (Over 3 months)?", options: ["Yes", "No"] }, { key: "pregnant", label: "Pregnant?", options: ["Yes", "No"] }, { key: "breastfeeding", label: "Breastfeeding?", options: ["Yes", "No"] }]
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

/* CLARINO BRAND COLOR PALETTE 
  Primary: #2D5A43 (Dark Silhouette Green)
  Secondary: #76B041 (Leaf Green)
  Soft: #F1F8F4 (Mint Background)
*/

.wizard-container {
  max-width: 650px;
  margin: 0 auto;
  padding: 40px 20px;
  font-family: 'Plus Jakarta Sans', sans-serif;
  color: #1F2F28; /* Dark forest grey */
}

/* CLINIC HEADER */
.clinic-header {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 40px;
}

.header-logo {
  height: 55px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(45, 90, 67, 0.15);
}

.header-text h1 {
  font-size: 1.4rem;
  font-weight: 800;
  margin: 0;
  color: #2D5A43; /* Brand Dark Green */
}

.header-text span {
  font-size: 0.7rem;
  color: #76B041; /* Brand Leaf Green */
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1.5px;
}

/* PROGRESS TRACKER */
.tracker-wrapper {
  margin-bottom: 30px;
}

.progress-bar {
  height: 8px;
  background: #E0EADD;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 10px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #2D5A43, #76B041);
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.steps-count {
  font-size: 0.8rem;
  color: #5C7067;
  font-weight: 600;
}

/* THE GLASS CARD */
.glass-card {
  background: white;
  border-radius: 28px;
  padding: 40px;
  box-shadow: 0 20px 50px rgba(45, 90, 67, 0.08); /* Green-tinted shadow */
  border: 1px solid #E9F0E8;
}

.step-title {
  font-size: 1.5rem;
  font-weight: 800;
  margin-bottom: 30px;
  color: #2D5A43;
}

/* UPLOAD ZONE */
.upload-zone {
  border: 2px dashed #B5C9BE;
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background: #F1F8F4; /* Soft Mint */
}

.upload-zone:hover {
  border-color: #76B041;
  background: #E8F5E9;
}

.upload-zone.has-image {
  padding: 10px;
  border-style: solid;
}

.icon-circle {
  width: 60px;
  height: 60px;
  background: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 15px;
  font-size: 1.5rem;
  color: #76B041;
  box-shadow: 0 10px 20px rgba(45, 90, 67, 0.1);
}

.preview-container {
  position: relative;
  border-radius: 16px;
  overflow: hidden;
}

.image-preview {
  width: 100%;
  max-height: 300px;
  object-fit: cover;
}

.overlay {
  position: absolute;
  bottom: 0;
  width: 100%;
  background: rgba(31, 47, 40, 0.7); /* Dark green-grey overlay */
  color: white;
  padding: 10px;
  font-size: 0.8rem;
  font-weight: 600;
}

/* FIELDS */
.fields-grid {
  display: grid;
  gap: 20px;
}

.field-item label {
  display: block;
  font-size: 0.85rem;
  font-weight: 700;
  margin-bottom: 8px;
  color: #2D5A43;
}

.select-wrapper select {
  width: 100%;
  padding: 14px;
  border-radius: 12px;
  border: 1px solid #D1DBD5;
  background: #fff;
  font-size: 0.95rem;
  appearance: none;
  transition: border-color 0.2s;
}

.select-wrapper select:focus {
  border-color: #76B041;
  outline: none;
  box-shadow: 0 0 0 4px rgba(118, 176, 65, 0.1);
}

/* BUTTONS */
.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 40px;
}

.btn-next,
.btn-submit {
  padding: 14px 32px;
  border-radius: 14px;
  background: #2D5A43;
  color: white;
  border: none;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-next:hover, .btn-submit:hover {
  background: #76B041;
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(118, 176, 65, 0.2);
}

.btn-ghost {
  background: none;
  border: none;
  color: #2D5A43;
  font-weight: 700;
  opacity: 0.7;
  cursor: pointer;
}

.btn-link {
  background: none;
  border: none;
  color: #5C7067;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  padding: 10px 15px;
  text-decoration: underline;
  transition: color 0.2s;
}

.btn-link:hover {
  color: #76B041;
}

.right-actions {
  display: flex;
  align-items: center;
  gap: 15px;
}

/* RESULT DASHBOARD */
.result-dashboard {
  background: white;
  border-radius: 28px;
  padding: 40px;
  box-shadow: 0 30px 60px rgba(45, 90, 67, 0.12);
}

.report-id {
  font-size: 0.75rem;
  font-weight: 800;
  color: #76B041;
  background: #F1F8F4;
  padding: 4px 12px;
  border-radius: 20px;
}

.condition-name {
  font-size: 2rem;
  font-weight: 800;
  color: #2D5A43;
  margin: 10px 0;
  border-left: 5px solid #76B041;
  padding-left: 15px;
}

.reasoning-text {
  color: #4A5D54;
  line-height: 1.6;
  font-size: 0.95rem;
}

.product-card-inner {
  background: #F1F8F4;
  padding: 15px;
  border-radius: 20px;
  text-align: center;
  border: 1px solid #E0EADD;
}

.product-shot {
  width: 100%;
  border-radius: 12px;
  margin-bottom: 12px;
}

.suitability-pill {
  display: inline-block;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 800;
  text-transform: uppercase;
}

.suitability-pill.safe {
  background: #DCF2E1;
  color: #1B4332;
}

.suitability-pill.unsafe {
  background: #FEE2E2;
  color: #991B1B;
}

.care-list li i {
  color: #76B041; /* Leaf Green Checkmarks */
}

.report-footer {
  border-top: 1px solid #E9F0E8;
  padding-top: 30px;
  margin-top: 30px;
}

.legal {
  font-size: 0.75rem;
  color: #5C7067;
  margin-bottom: 20px;
}

/* RESPONSIVE */
@media (max-width: 600px) {
  .report-grid {
    grid-template-columns: 1fr;
  }
  .product-recommendation {
    order: -1;
  }
  .glass-card, .result-dashboard {
    padding: 25px;
  }
}
</style>