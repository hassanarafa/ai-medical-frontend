<template>
  <div class="wizard-container">

    <!-- PROGRESS BAR -->
    <div class="progress-wrapper">
      <div class="progress" :style="{ width: progressWidth + '%' }"></div>
    </div>

    <!-- STEPPER -->
    <div class="stepper">
      <div v-for="(step, index) in steps" :key="index" class="stepper-item"
        :class="{ active: index === currentStep, done: index < currentStep }">
        <span>{{ index + 1 }}</span>
      </div>
    </div>

    <!-- TITLE -->
    <h2 class="title">{{ steps[currentStep].title }}</h2>

    <!-- MAIN CARD -->
    <div class="card">

      <!-- STEP CONTENT -->
      <transition name="fade-slide" mode="out-in">
        <div :key="currentStep" class="step">

          <!-- STEP 1: IMAGE UPLOAD -->
          <template v-if="currentStep === 0">
            <label class="label">Upload Acne Image</label>

            <div class="upload-box" @click="$refs.imageInput.click()">
              <input ref="imageInput" type="file" accept="image/*" hidden @change="onFileChange" />

              <div v-if="image" class="preview">
                <img :src="image" />
                <span class="change-photo">Change image</span>
              </div>

              <div v-else class="upload-placeholder">
                <i class="ph ph-camera"></i>
                <strong>Upload skin image</strong>
                <small>PNG or JPG • Max 5MB</small>
              </div>
            </div>
          </template>

          <!-- OTHER STEPS -->
          <template v-else>
            <div v-for="field in currentFields" :key="field.key" class="field">
              <label class="label">{{ field.label }}</label>
              <select v-model="form[field.key]" class="input">
                <option disabled value="">Select...</option>
                <option v-for="opt in field.options" :key="opt">
                  {{ opt }}
                </option>
              </select>
            </div>
          </template>

          <!-- FINAL STEP -->
          <div v-if="currentStep === lastStep" class="final-step">
            <button class="btn-diagnose" @click="submitForm" :disabled="loading">
              <span v-if="loading">🧬 Analyzing Skin Texture...</span>
              <span v-else>Diagnose Skin Condition</span>
            </button>
          </div>

        </div>
      </transition>

      <!-- FOOTER BUTTONS -->
      <div class="step-buttons">
        <button v-if="currentStep > 0" class="btn-outline" @click="prevStep">
          Back
        </button>

        <button v-if="steps[currentStep].optional && currentStep < lastStep" class="btn-skip" @click="skipStep">
          Skip
        </button>

        <button v-if="currentStep < lastStep" class="btn-primary" :disabled="currentStep === 0 && !image"
          @click="nextStep">
          Next
        </button>
      </div>

    </div>

    <!-- RESULT -->
    <transition name="fade">
      <div v-if="result" class="result-card">
        <h3>🩺 Diagnosis Result</h3>
        <p class="result-note">AI-based preliminary analysis</p>

        <div class="result-content" v-if="result.disease">
          <p><strong>Disease:</strong> {{ result.disease }}</p>
          <p><strong>Confidence:</strong> {{ result.confidence }}</p>

          <p class="section-title">Description</p>
          <p>{{ result.description }}</p>

          <p class="section-title">Recommendations</p>
          <ul>
            <li v-for="(rec, i) in result.recommendations" :key="i">
              {{ rec }}
            </li>
          </ul>

          <p class="disclaimer">
            ⚠ {{ result.medicalAdvice }}
          </p>
        </div>

        <!-- fallback for old style or raw JSON -->
        <pre v-else>{{ result }}</pre>
      </div>
    </transition>

  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      currentStep: 0,
      steps: [
        { title: "Upload Image", optional: false },
        { title: "Your Profile", optional: false },
        { title: "Skin & Food", optional: false },
        { title: "Pain & Sensitivity", optional: false },
        { title: "Appearance Details", optional: true },
        { title: "Location & Allergy", optional: true },
        { title: "Medical Conditions", optional: true },
        { title: "Submit" }
      ],
      form: {
        gender: "",
        age: "",
        skinType: "",
        fastFood: "",
        timeSensitive: "",
        painful: "",
        pus: "",
        redness: "",
        location: "",
        allergy: "",
        longDuration: "",
        pregnant: "",
        breastfeeding: ""
      },
      image: null,      // Used for preview (base64)
      rawFile: null,    // The actual File object for the backend
      result: null,
      loading: false    // Added loading state
    };
  },

  computed: {
    lastStep() {
      return this.steps.length - 1;
    },
    progressWidth() {
      return ((this.currentStep + 1) / this.steps.length) * 100;
    },
    currentFields() {
      const map = [
        [],
        [
          { key: "gender", label: "Gender", options: ["Male", "Female"] },
          { key: "age", label: "Age", options: ["11-14", "14-18", "18-21", "21-30"] }
        ],
        [
          { key: "skinType", label: "Skin Type", options: ["Normal", "Oily", "Dry", "Mixed"] },
          { key: "fastFood", label: "Eat Fast Food?", options: ["Yes", "No"] }
        ],
        [
          { key: "timeSensitive", label: "Acne worse at specific time?", options: ["Yes", "No"] },
          { key: "painful", label: "Painful?", options: ["Yes", "No"] }
        ],
        [
          { key: "pus", label: "Contains pus?", options: ["Yes", "No"] },
          { key: "redness", label: "Redness?", options: ["Yes", "No"] }
        ],
        [
          { key: "location", label: "Location", options: ["T-Zone", "Forehead", "Cheeks", "Chin"] },
          { key: "allergy", label: "Allergy?", options: ["None", "Medicine", "Cosmetics", "Food"] }
        ],
        [
          { key: "longDuration", label: "Stays long?", options: ["Yes", "No"] },
          { key: "pregnant", label: "Pregnant?", options: ["Yes", "No"] },
          { key: "breastfeeding", label: "Breastfeeding?", options: ["Yes", "No"] }
        ]
      ];
      return map[this.currentStep] || [];
    }
  },

  methods: {
    nextStep() {
      if (this.currentStep < this.lastStep) this.currentStep++;
    },
    prevStep() {
      if (this.currentStep > 0) this.currentStep--;
    },
    skipStep() {
      this.nextStep();
    },
    async onFileChange(e) {
      const file = e.target.files[0];
      if (!file) return;

      // 1. Create a preview for the UI
      const reader = new FileReader();
      reader.onload = () => (this.image = reader.result);
      reader.readAsDataURL(file);

      // 2. Resize the image for the Backend (Vercel)
      this.rawFile = await this.compressImage(file);
    },

    compressImage(file) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = (event) => {
          const img = new Image();
          img.src = event.target.result;
          img.onload = () => {
            const canvas = document.createElement('canvas');
            let width = img.width;
            let height = img.height;

            // Max width/height of 1600px is plenty for Gemini
            const MAX_SIZE = 1600;
            if (width > height) {
              if (width > MAX_SIZE) {
                height *= MAX_SIZE / width;
                width = MAX_SIZE;
              }
            } else {
              if (height > MAX_SIZE) {
                width *= MAX_SIZE / height;
                height = MAX_SIZE;
              }
            }

            canvas.width = width;
            canvas.height = height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0, width, height);

            // Convert back to file at 70% quality (drastically reduces MBs)
            canvas.toBlob((blob) => {
              resolve(new File([blob], file.name, { type: 'image/jpeg' }));
            }, 'image/jpeg', 0.7);
          };
        };
      });
    },
    async submitForm() {
      this.loading = true;
      this.result = null;

      try {
        const formData = new FormData();
        if (!this.rawFile) {
          alert("Please upload an image in Step 1");
          this.currentStep = 0;
          return;
        }
        formData.append("image", this.rawFile);
        formData.append("user_answers", JSON.stringify(this.form));
        const res = await axios.post("https://aimedicalrepo-production.up.railway.app/analyze", formData, {
        // const res = await axios.post("https://ai-medical-repo.vercel.app/api/analyze", formData, {
          headers: { "Content-Type": "multipart/form-data" }
        });

        const aiResponse = res.data;
        this.result = {
          disease: aiResponse.diagnosis,
          confidence: "High Definition AI Analysis",
          description: aiResponse.reasoning,
          recommendations: [
            `Clarino Suitability: ${aiResponse.suitability}`,
            aiResponse.clinical_note || "Maintain routine skin hygiene."
          ],
          medicalAdvice: "This result is AI-generated and for educational purposes only."
        };

      } catch (err) {
        console.error("Error:", err);
        alert(err.response?.status === 413 ? "Image too large for Vercel (4.5MB limit)" : "Server error.");
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style>
/* PAGE */
.wizard-container {
  max-width: 560px;
  margin: auto;
  padding: 30px 16px;
  font-family: "Poppins", sans-serif;
}

/* PROGRESS */
.progress-wrapper {
  height: 7px;
  background: #e5e7eb;
  border-radius: 10px;
  overflow: hidden;
}

.progress {
  height: 100%;
  background: linear-gradient(90deg, #4b6cff, #a05bff);
  transition: width 0.4s ease;
}

/* STEPPER */
.stepper {
  display: flex;
  justify-content: space-between;
  margin: 16px 0;
}

.stepper-item {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #6b7280;
}

.stepper-item.active {
  background: linear-gradient(135deg, #4b6cff, #a05bff);
  color: white;
  transform: scale(1.1);
}

.stepper-item.done {
  background: #22c55e;
  color: white;
}

/* CARD */
.card {
  background: white;
  padding: 26px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

/* TITLES */
.title {
  text-align: center;
  margin-bottom: 16px;
  font-weight: 700;
}

/* STEP */
.step {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* INPUT */
.label {
  font-size: 0.85rem;
  font-weight: 600;
}

.input {
  padding: 12px;
  border-radius: 12px;
  border: 1px solid #d1d5db;
  background: #f9fafb;
  appearance: none;
}

/* UPLOAD */
.upload-box {
  border: 2px dashed #b9c4ff;
  border-radius: 16px;
  padding: 30px;
  text-align: center;
  cursor: pointer;
}

.upload-placeholder small {
  opacity: 0.7;
}

.preview img {
  width: 180px;
  border-radius: 16px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
}

.change-photo {
  margin-top: 8px;
  display: block;
  font-size: 0.8rem;
  color: #4b6cff;
}

/* BUTTONS */
.step-buttons {
  margin-top: 24px;
  display: flex;
  gap: 10px;
  justify-content: center;
}

button {
  padding: 12px 22px;
  border-radius: 12px;
  font-weight: 600;
  cursor: pointer;
}

.btn-primary {
  background: linear-gradient(90deg, #4b6cff, #855cff);
  color: white;
  border: none;
}

.btn-outline {
  border: 1px solid #4b6cff;
  color: #4b6cff;
  background: white;
}

.btn-skip {
  background: #e5e7eb;
}

.btn-diagnose {
  width: 100%;
  background: linear-gradient(90deg, #4b6cff, #a05bff);
  color: white;
  border: none;
  padding: 14px;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* RESULT */
.result-card {
  margin-top: 24px;
  background: white;
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
}

.result-note {
  font-size: 0.8rem;
  color: #6b7280;
}

.result-content p {
  margin-bottom: 8px;
  font-size: 0.9rem;
}

.section-title {
  font-weight: 600;
  margin-top: 12px;
}

.result-content ul {
  padding-left: 18px;
  font-size: 0.85rem;
}

.disclaimer {
  margin-top: 14px;
  font-size: 0.75rem;
  color: #6b7280;
}

/* ANIMATION */
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.fade-slide-enter-active {
  transition: 0.3s ease;
}

.fade-slide-leave-active {
  transition: 0.25s ease;
  opacity: 0;
}
</style>
