<template>
    <div class="landing-container">
        <nav class="navbar">
            <div class="nav-content">
                <div class="logo-area">
                    <img :src="clarinoLogo" alt="Clarino" class="nav-logo" />
                    <div class="logo-text">
                        <span class="brand-name">Clarino AI</span>
                        <span class="brand-tag">Dermatological Systems</span>
                    </div>
                </div>
                <div class="nav-links">
                    <a href="#features">Technology</a>
                    <a href="#how-it-works">Process</a>
                    <a href="#faq">FAQ</a>
                    <button @click="$emit('start-wizard')" class="btn-cta-sm">Launch Scan</button>
                </div>
            </div>
        </nav>

        <header class="hero-section">
            <div class="hero-content">
                <div class="pill-badge">⚡ Powered by Next-Gen Computer Vision</div>
                <h1>Instant Clinical Skin Intelligence.</h1>
                <p class="hero-subtitle">
                    Stop guessing your skincare routine. Clarino leverages advanced convolutional neural networks
                    trained alongside world-class dermatologists to pinpoint issues and safely match products in under
                    60 seconds.
                </p>
                <div class="hero-actions">
                    <button @click="$emit('start-wizard')" class="btn-cta-lg">
                        Start Free AI Diagnosis <i class="ph ph-sparkle"></i>
                    </button>
                    <a href="#features" class="btn-secondary">Explore the Science</a>
                </div>
                <div class="hero-metrics">
                    <div class="metric-item"><strong>98.4%</strong> <span>AI Accuracy Matrix</span></div>
                    <div class="metric-item"><strong>&lt; 1 Min</strong> <span>Instant Processing</span></div>
                    <div class="metric-item"><strong>100%</strong> <span>HIPAA Compliant Privacy</span></div>
                </div>
            </div>
        </header>

        <section id="features" class="features-section">
            <div class="section-header">
                <h2>Engineered for Skin Precision</h2>
                <p>A multi-layered diagnostic engine that considers raw visual data alongside your lifestyle biomarkers.
                </p>
            </div>

            <div class="features-grid">
                <div class="feature-card">
                    <div class="icon-box"><i class="ph ph-scan"></i></div>
                    <h3>Macro Photo Analysis</h3>
                    <p>Scans cellular patterns, depth vectors, and localized redness markers to distinguish simple
                        congestion from persistent dermal conditions.</p>
                </div>
                <div class="feature-card">
                    <div class="icon-box"><i class="ph ph-heart-beat"></i></div>
                    <h3>Biomarker Correlation</h3>
                    <p>Factors in structural questions regarding diet, location mapping, hormonal sensitivity triggers,
                        and product allergens.</p>
                </div>
                <div class="feature-card">
                    <div class="icon-box"><i class="ph ph-shield-check"></i></div>
                    <h3>Clarino Suitability Engine</h3>
                    <p>Cross-references identified complications with safe topical solutions, instantly generating an
                        active warning label or an green-lit routine blueprint.</p>
                </div>
            </div>
        </section>

        <section id="how-it-works" class="how-it-works">
            <div class="section-header">
                <h2>Three Steps to Absolute Clarity</h2>
            </div>
            <div class="steps-timeline">
                <div class="timeline-step">
                    <div class="step-num">01</div>
                    <h4>Capture Visuals</h4>
                    <p>Upload a clear, macro photo of the targeted area under high-fidelity lighting conditions using
                        your desktop or smartphone camera.</p>
                </div>
                <div class="timeline-step">
                    <div class="step-num">02</div>
                    <h4>Biometric Questionnaire</h4>
                    <p>Fill out a rapid 7-part lifestyle map assessing items like skin microclimates, dietary
                        sensitivities, and systemic history markers.</p>
                </div>
                <div class="timeline-step">
                    <div class="step-num">03</div>
                    <h4>Review Clinical Dashboard</h4>
                    <p>Obtain an instant interactive breakdown detailing your specific classification structure, Clarino
                        suitability index, and custom care layout.</p>
                </div>
            </div>
        </section>

        <section id="faq" class="faq-section">
            <div class="section-header">
                <h2>Frequently Addressed Concepts</h2>
            </div>
            <div class="faq-accordion">
                <div v-for="(item, idx) in faqs" :key="idx" class="faq-item" :class="{ active: activeFaq === idx }"
                    @click="toggleFaq(idx)">
                    <div class="faq-question">
                        <h4>{{ item.q }}</h4>
                        <span class="faq-icon"><i class="ph"
                                :class="activeFaq === idx ? 'ph-minus' : 'ph-plus'"></i></span>
                    </div>
                    <div class="faq-answer" v-show="activeFaq === idx">
                        <p>{{ item.a }}</p>
                    </div>
                </div>
            </div>
        </section>

        <section class="cta-banner">
            <div class="banner-box">
                <h2>Ready to map your real skin profile?</h2>
                <p>Access our comprehensive clinical evaluation system for free today. No subscriptions required.</p>
                <button @click="$emit('start-wizard')" class="btn-cta-white">Launch Diagnostics Now</button>
            </div>
        </section>

        <footer class="app-footer">
            <div class="footer-content">
                <p class="footer-copyright">&copy; 2026 Clarino Systems Inc. All rights reserved.</p>
                <div class="footer-legal">
                    <span>Privacy Policy</span> | <span>Terms of Deployment</span> | <span>Medical Disclaimer</span>
                </div>
            </div>
        </footer>
    </div>
</template>

<script>
import clarinoLogo from "../assets/logo.png";

export default {
    emits: ["start-wizard"],
    data() {
        return {
            clarinoLogo,
            activeFaq: null,
            faqs: [
                { q: "Is this tool an official medical replacement?", a: "No. Clarino provides an artificial intelligence assessment using clinical deep learning layers. It offers educational clarity and product compatibility guidelines, but doesn't override structural validation from licensed practitioners." },
                { q: "How is my high-resolution visual data stored?", a: "Photos are immediately processed server-side through advanced encryption channels and are not cached, archived, or shared with external third-party cosmetic databases." },
                { q: "What conditions can the AI engine accurately screen?", a: "The analysis framework maps regular dermal attributes, variations in targeted acne types (pustules, nodules, cystic), textural shifts, contact irritation signatures, and hydration variations." }
            ]
        };
    },
    methods: {
        toggleFaq(index) {
            this.activeFaq = this.activeFaq === index ? null : index;
        }
    }
};
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap');

/* GLOBAL CONTAINER SETUP */
.landing-container {
    background-color: #F4FBF7;
    font-family: 'Plus Jakarta Sans', sans-serif;
    color: #1F2F28;
    min-height: 100vh;
    width: 100%;
    overflow-x: hidden;
    /* Guards against horizontal jitter rows */
}

/* NAVBAR - Full Width Row Track */
.navbar {
    position: sticky;
    top: 0;
    width: 100%;
    background: rgba(244, 251, 247, 0.85);
    backdrop-filter: blur(12px);
    z-index: 100;
    border-bottom: 1px solid #E1EFE8;
}

.nav-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 15px 24px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.logo-area {
    display: flex;
    align-items: center;
    gap: 10px;
}

.nav-logo {
    height: 40px;
    border-radius: 8px;
}

.brand-name {
    display: block;
    font-weight: 800;
    font-size: 1.1rem;
    color: #2D5A43;
    line-height: 1;
}

.brand-tag {
    font-size: 0.65rem;
    color: #76B041;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.nav-links {
    display: flex;
    align-items: center;
    gap: 30px;
}

.nav-links a {
    text-decoration: none;
    color: #5C7067;
    font-weight: 600;
    font-size: 0.9rem;
    transition: color 0.2s;
}

.nav-links a:hover {
    color: #2D5A43;
}

/* HERO - Full Viewport Range */
.hero-section {
    width: 100%;
    box-sizing: border-box;
    padding: 100px 24px 80px;
    text-align: center;
}

.pill-badge {
    display: inline-block;
    background: #E2F3EB;
    color: #2D5A43;
    padding: 6px 16px;
    border-radius: 30px;
    font-size: 0.8rem;
    font-weight: 700;
    margin-bottom: 25px;
}

.hero-section h1 {
    font-size: 3.5rem;
    font-weight: 800;
    color: #2D5A43;
    line-height: 1.1;
    margin: 0 auto 20px;
    letter-spacing: -1px;
    max-width: 900px;
}

.hero-subtitle {
    max-width: 750px;
    margin: 0 auto 40px;
    font-size: 1.15rem;
    color: #5C7067;
    line-height: 1.6;
}

.hero-actions {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 20px;
    margin-bottom: 60px;
}

/* BUTTONS */
.btn-cta-lg {
    background: #2D5A43;
    color: white;
    border: none;
    padding: 18px 36px;
    border-radius: 14px;
    font-weight: 700;
    font-size: 1.05rem;
    cursor: pointer;
    transition: all 0.2s;
    box-shadow: 0 10px 25px rgba(45, 90, 67, 0.15);
}

.btn-cta-lg:hover {
    background: #76B041;
    transform: translateY(-2px);
}

.btn-cta-sm {
    background: #2D5A43;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 10px;
    font-weight: 700;
    font-size: 0.85rem;
    cursor: pointer;
    transition: 0.2s;
}

.btn-cta-sm:hover {
    background: #76B041;
}

.btn-secondary {
    color: #2D5A43;
    text-decoration: none;
    font-weight: 700;
    font-size: 0.95rem;
    border-bottom: 2px solid #C2E5D3;
    padding-bottom: 4px;
    transition: all 0.2s;
}

.btn-secondary:hover {
    border-color: #2D5A43;
}

.hero-metrics {
    display: flex;
    justify-content: center;
    gap: 80px;
    border-top: 1px solid #E1EFE8;
    padding-top: 40px;
    max-width: 1200px;
    margin: 0 auto;
}

.metric-item strong {
    display: block;
    font-size: 2rem;
    color: #2D5A43;
    font-weight: 800;
}

.metric-item span {
    font-size: 0.85rem;
    color: #76B041;
    font-weight: 600;
}

/* STRIPED INTERACTIVE GRID BLOCKS - Allowing true full-screen canvas bleed */
.features-section,
.how-it-works,
.faq-section {
    width: 100%;
    box-sizing: border-box;
    padding: 90px 24px;
}

/* Optional Zebra Striping background rows for design depth */
.features-section {
    background-color: #ffffff;
    border-top: 1px solid #E1EFE8;
    border-bottom: 1px solid #E1EFE8;
}

.how-it-works {
    background-color: #F4FBF7;
}

.faq-section {
    background-color: #ffffff;
    border-top: 1px solid #E1EFE8;
}

/* Internal contents are max-bounded & completely safe */
.section-header,
.features-grid,
.steps-timeline,
.faq-accordion {
    max-width: 1200px;
    margin: 0 auto;
}

.section-header {
    text-align: center;
    margin-bottom: 60px;
}

.section-header h2 {
    font-size: 2.4rem;
    font-weight: 800;
    color: #2D5A43;
    letter-spacing: -0.5px;
}

.section-header p {
    color: #5C7067;
    max-width: 600px;
    margin: 12px auto 0;
    font-size: 1.05rem;
}

.features-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 32px;
}

.feature-card {
    background: white;
    padding: 40px 30px;
    border-radius: 24px;
    border: 1px solid #E1EFE8;
    box-shadow: 0 12px 35px rgba(45, 90, 67, 0.03);
}

.icon-box {
    background: #C2E5D3;
    width: 52px;
    height: 52px;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.6rem;
    color: #2D5A43;
    margin-bottom: 24px;
}

.feature-card h3 {
    font-size: 1.3rem;
    font-weight: 700;
    color: #2D5A43;
    margin-bottom: 12px;
}

.feature-card p {
    font-size: 0.98rem;
    color: #5C7067;
    line-height: 1.6;
}

/* STEPS TIMELINE */
.steps-timeline {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 40px;
    position: relative;
}

.timeline-step {
    position: relative;
}

.step-num {
    font-size: 3.5rem;
    font-weight: 800;
    color: #C2E5D3;
    line-height: 1;
    margin-bottom: 12px;
}

.timeline-step h4 {
    font-size: 1.25rem;
    font-weight: 700;
    color: #2D5A43;
    margin-bottom: 8px;
}

.timeline-step p {
    font-size: 0.98rem;
    color: #5C7067;
    line-height: 1.6;
}

/* ACCORDION FAQ */
.faq-accordion {
    max-width: 800px;
    /* Gives standard proportional spacing balance to accordions */
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.faq-item {
    background: white;
    border: 1px solid #E1EFE8;
    border-radius: 16px;
    padding: 24px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.faq-item:hover {
    border-color: #76B041;
    box-shadow: 0 4px 15px rgba(118, 176, 65, 0.05);
}

.faq-question {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.faq-question h4 {
    font-size: 1.1rem;
    font-weight: 700;
    margin: 0;
    color: #2D5A43;
}

.faq-icon {
    color: #76B041;
    font-size: 1.1rem;
}

.faq-answer {
    margin-top: 16px;
    padding-top: 16px;
    border-top: 1px solid #F0F6F3;
    font-size: 0.98rem;
    color: #5C7067;
    line-height: 1.6;
}

/* CTA CONVERSION BANNER */
.cta-banner {
    width: 100%;
    box-sizing: border-box;
    padding: 20px 24px 100px;
    background-color: #ffffff;
}

.banner-box {
    max-width: 1200px;
    margin: 0 auto;
    background: linear-gradient(135deg, #2D5A43, #1A3A2B);
    color: white;
    border-radius: 32px;
    padding: 80px 40px;
    text-align: center;
    box-shadow: 0 20px 45px rgba(45, 90, 67, 0.18);
}

.banner-box h2 {
    font-size: 2.5rem;
    font-weight: 800;
    margin-bottom: 16px;
    letter-spacing: -0.5px;
}

.banner-box p {
    max-width: 550px;
    margin: 0 auto 36px;
    font-size: 1.1rem;
    opacity: 0.9;
    line-height: 1.5;
}

.btn-cta-white {
    background: white;
    color: #2D5A43;
    border: none;
    padding: 18px 40px;
    border-radius: 14px;
    font-weight: 700;
    font-size: 1rem;
    cursor: pointer;
    transition: 0.2s;
}

.btn-cta-white:hover {
    background: #C2E5D3;
    transform: translateY(-2px);
}

/* FOOTER - Anchored Edge-to-Edge with Deep Dark Premium Contrast */
.app-footer {
    width: 100%;
    background-color: #1A2821;
    border-top: 1px solid #23372D;
    box-sizing: border-box;
}

.footer-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 40px 24px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 0.88rem;
    color: #8A9D94;
}

.footer-legal span {
    cursor: pointer;
    transition: color 0.2s;
    margin-left: 15px;
}

.footer-legal span:hover {
    color: #ffffff;
}

/* MOBILE RESPONSIVENESS */
@media(max-width: 768px) {
    .nav-links {
        display: none;
    }

    .hero-section {
        padding: 60px 20px 50px;
    }

    .hero-section h1 {
        font-size: 2.2rem;
    }

    .hero-actions {
        flex-direction: column;
        width: 100%;
        gap: 15px;
    }

    .btn-cta-lg {
        width: 100%;
    }

    .hero-metrics {
        flex-direction: column;
        gap: 30px;
    }

    .features-section,
    .how-it-works,
    .faq-section {
        padding: 60px 20px;
    }

    .section-header h2 {
        font-size: 1.8rem;
    }

    .features-grid,
    .steps-timeline {
        grid-template-columns: 1fr;
        gap: 30px;
    }

    .banner-box {
        padding: 50px 20px;
    }

    .banner-box h2 {
        font-size: 1.8rem;
    }

    .footer-content {
        flex-direction: column;
        gap: 20px;
        text-align: center;
        padding: 40px 20px;
    }

    .footer-legal span {
        margin: 0 8px;
    }
}
</style>