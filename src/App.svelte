<script>
  import { Router, Route } from 'svelte-routing';
  import Plan from './routes/Plan.svelte';
  import { plans } from './data.js';
  import emailjs from '@emailjs/browser';

  let selectedPlan = null;
  let showBookingForm = false;
  let name = "";
  let email = "";
  let problem = "";
  let submitted = false;
  let loading = false;
  let errorMessage = "";
  let selectedDate = "";
  let selectedTime = "";
  let preferredContact = "email";
  let openFaqIndex = null;

  const EMAIL_SERVICE_ID = import.meta.env.VITE_EMAIL_SERVICE_ID;
  const EMAIL_TEMPLATE_ID = import.meta.env.VITE_EMAIL_TEMPLATE_ID;
  const EMAIL_PUBLIC_KEY = import.meta.env.VITE_EMAIL_PUBLIC_KEY;

  const faqs = [
    {
      question: "How does it work?",
      answer: "Simply choose a plan, book a session, and connect with your listener at the scheduled time. All sessions are confidential and tailored to your needs."
    },
    {
      question: "Are the sessions confidential?",
      answer: "Yes, all sessions are 100% confidential. We follow strict privacy guidelines to ensure your information is secure."
    },
    {
      question: "Can I change my plan?",
      answer: "Yes, you can upgrade or downgrade your plan at any time. Contact our support team for assistance."
    },
    {
      question: "What if I need to cancel?",
      answer: "You can cancel or reschedule your session up to 24 hours before the appointment without any charges."
    }
  ];

  function selectPlan(plan) {
    selectedPlan = plan;
    showBookingForm = true;
  }

  async function handleSubmit(event) {
    event.preventDefault();
    loading = true;
    errorMessage = "";

    try {
      const templateParams = {
        to_name: name,
        to_email: email,
        plan_name: selectedPlan.title,
        plan_price: selectedPlan.price,
        plan_features: selectedPlan.features.join('\n• '),
        plan_description: selectedPlan.description,
        message: problem,
        booking_date: selectedDate,
        booking_time: selectedTime,
        preferred_contact: preferredContact,
        support_email: "support@listenwell.com",
        website_url: "https://listenwell.com",
        session_type: selectedPlan.type,
        session_duration: selectedPlan.features[0].split('-')[0].trim()
      };

      await emailjs.send(
        EMAIL_SERVICE_ID,
        EMAIL_TEMPLATE_ID,
        templateParams,
        EMAIL_PUBLIC_KEY
      );

      submitted = true;
      showBookingForm = false;
    } catch (error) {
      errorMessage = "Sorry, something went wrong. Please try again.";
      console.error(error);
    } finally {
      loading = false;
    }
  }

  function startOver() {
    selectedPlan = null;
    showBookingForm = false;
    submitted = false;
    name = "";
    email = "";
    problem = "";
    errorMessage = "";
    selectedDate = "";
    selectedTime = "";
    preferredContact = "email";
  }

  function toggleFaq(index) {
    openFaqIndex = openFaqIndex === index ? null : index;
  }

  let menuOpen = false;

  function toggleMenu() {
    menuOpen = !menuOpen;
    const navLinks = document.querySelector('.nav-links');
    navLinks.classList.toggle('active');
  }

  document.querySelector('.menu-toggle')?.addEventListener('click', toggleMenu);
</script>

<Router>
  <Route path="/">
    <nav>
      <div class="logo">
        <svg xmlns="http://www.w3.org/2000/svg" width="35" height="35" viewBox="0 0 24 24"><path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 21c.757.667 1.424 1 2 1c2 0 3-1 3-3c0-1.333.667-2.667 2-4c1.267-1.267 2-3.067 2-5a7 7 0 0 0-14 0m11 0a4 4 0 1 0-8 0M3 20l5-6l1 4l5-6"/></svg>
        <span>ListenWell</span>
      </div>
      <div class="nav-links">
        <a href="/#about">About</a>
        <a href="/#plans">Plans</a>
        <a href="/#faq">FAQ</a>
      </div>
      <button class="menu-toggle" aria-label="Toggle menu">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <line x1="3" y1="12" x2="21" y2="12"/>
          <line x1="3" y1="6" x2="21" y2="6"/>
          <line x1="3" y1="18" x2="21" y2="18"/>
        </svg>
      </button>
    </nav>

    <header>
      <h1>Someone to Talk To</h1>
      <p>Professional listening service. No judgment, just support.</p>
      <div class="header-stats">
        <div class="stat">
          <span class="number">1000+</span>
          <span class="label">Sessions Completed</span>
        </div>
        <div class="stat">
          <span class="number">98%</span>
          <span class="label">Satisfaction Rate</span>
        </div>
        <div class="stat">
          <span class="number">24/7</span>
          <span class="label">Support Available</span>
        </div>
      </div>
    </header>

    <section class="video-section" id="about">
      <div class="video-container">
        <iframe 
          width="560" 
          height="315" 
          src="https://www.youtube.com/embed/eIho2S0ZahI" 
          title="ListenWell Introduction" 
          frameborder="0" 
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
          allowfullscreen>
        </iframe>
      </div>
      <div class="video-content">
        <h2>Why Choose ListenWell?</h2>
        <p>We provide a safe space for you to express yourself freely. Our professional listeners are here to support you through life's challenges.</p>
        <ul>
          <li>Professional listeners</li>
          <li>Confidential sessions</li>
          <li>Flexible scheduling</li>
        </ul>
      </div>
    </section>

    <section class="plans-section" id="plans">
      <h2>Choose Your Plan</h2>
      <div class="plans">
        {#each plans as plan}
          <div class="plan {plan.popular ? 'popular' : ''}">
            <div class="plan-header">
              <h2>{plan.title}</h2>
              <span class="icon">
                {#if plan.type === 'text'}
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
                  </svg>
                {:else if plan.type === 'video'}
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <polygon points="23 7 16 12 23 17 23 7"/>
                    <rect x="1" y="5" width="15" height="14" rx="2" ry="2"/>
                  </svg>
                {:else}
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/>
                    <circle cx="9" cy="7" r="4"/>
                    <path d="M23 21v-2a4 4 0 0 0-3-3.87"/>
                    <path d="M16 3.13a4 4 0 0 1 0 7.75"/>
                  </svg>
                {/if}
              </span>
            </div>
            {#if plan.title !== 'Free Support'}
              <div class="price">$0.1 <span class="period">/month</span></div>
            {:else}
              <div class="price">Free</div>
            {/if}
            <p class="description">{plan.description}</p>
            <ul>
              {#each plan.features as feature}
                <li>{feature}</li>
              {/each}
            </ul>
            <a href="/plan/{plan.title.toLowerCase().replace(/\s+/g, '-')}" class="primary">Choose Plan</a>
          </div>
        {/each}
      </div>
    </section>

    <section class="faq-section" id="faq">
      <h2>Frequently Asked Questions</h2>
      <div class="faq-container">
        {#each faqs as faq, i}
          <div class="faq-item {openFaqIndex === i ? 'open' : ''}">
            <button class="faq-question" on:click={() => toggleFaq(i)}>
              <span>{faq.question}</span>
              <svg class="icon {openFaqIndex === i ? 'rotate' : ''}" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M6 9l6 6 6-6"/>
              </svg>
            </button>
            <div class="faq-answer">
              <p>{faq.answer}</p>
            </div>
          </div>
        {/each}
      </div>
    </section>

    <footer>
      <div class="footer-content">
        <div class="footer-section">
          <div class="logo">
            <svg xmlns="http://www.w3.org/2000/svg" width="35" height="35" viewBox="0 0 24 24"><path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 21c.757.667 1.424 1 2 1c2 0 3-1 3-3c0-1.333.667-2.667 2-4c1.267-1.267 2-3.067 2-5a7 7 0 0 0-14 0m11 0a4 4 0 1 0-8 0M3 20l5-6l1 4l5-6"/></svg>
            <span>ListenWell</span>
          </div>
          <p>Professional listening service. No judgment, just support.</p>
          <div class="social-links">
            <a href="https://peerlist.io/mdanassaif" class="peerlist" target="_blank" rel="noopener noreferrer">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path fill="currentColor" d="M12 0C2.667 0 0 2.667 0 12s2.673 12 12 12s12-2.667 12-12S21.327 0 12 0zm8.892 20.894c-1.57 1.569-4.247 2.249-8.892 2.249s-7.322-.68-8.892-2.25C1.735 19.522 1.041 17.3.89 13.654A39.74 39.74 0 0 1 .857 12c0-1.162.043-2.201.13-3.13c.177-1.859.537-3.278 1.106-4.366c.284-.544.62-1.006 1.013-1.398s.854-.729 1.398-1.013C5.592 1.524 7.01 1.164 8.87.988C9.799.9 10.838.858 12 .858c4.645 0 7.322.68 8.892 2.248c1.569 1.569 2.25 4.246 2.25 8.894s-.681 7.325-2.25 8.894zM20.538 3.46C19.064 1.986 16.51 1.357 12 1.357c-4.513 0-7.067.629-8.54 2.103C1.986 4.933 1.357 7.487 1.357 12c0 4.511.63 7.065 2.105 8.54C4.936 22.014 7.49 22.643 12 22.643s7.064-.629 8.538-2.103c1.475-1.475 2.105-4.029 2.105-8.54s-.63-7.065-2.105-8.54zM14.25 16.49a6.097 6.097 0 0 1-2.442.59v2.706H10.45v.357H6.429V5.57h.357V4.214h5.676c3.565 0 6.467 2.81 6.467 6.262c0 2.852-1.981 5.26-4.68 6.013zm-1.788-8.728H10.45v5.428h2.011c1.532 0 2.802-1.2 2.802-2.714s-1.27-2.714-2.802-2.714zm.901 4.351c.117-.239.186-.502.186-.78c0-1.01-.855-1.857-1.945-1.857h-.296V8.62h1.154c1.09 0 1.945.847 1.945 1.857c0 .705-.422 1.323-1.044 1.637zm4.104 1.493c.043-.063.083-.129.123-.194a5.653 5.653 0 0 0 .526-1.103a5.56 5.56 0 0 0 .11-.362c.02-.076.042-.15.06-.227a5.58 5.58 0 0 0 .073-.41c.01-.068.025-.134.032-.203c.024-.207.038-.417.038-.63c0-3.198-2.687-5.763-5.967-5.763H7.286v14.572h4.022v-3.048h1.154c1.43 0 2.747-.488 3.778-1.303a5.92 5.92 0 0 0 .46-.406c.035-.034.066-.07.1-.105c.107-.11.21-.22.308-.337c.044-.053.084-.108.126-.162c.081-.104.16-.21.233-.319zm-5.005 1.775H10.45v3.048H8.143V5.57h4.319c2.837 0 5.11 2.211 5.11 4.905s-2.273 4.905-5.11 4.905z"/></svg>
            </a>

           
            <a href="https://github.com/mdanassaif" class="github" target="_blank" rel="noopener noreferrer">
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
              </svg>
            </a>
            <a href="https://twitter.com/mdanassaif" class="twitter" target="_blank" rel="noopener noreferrer">
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M23 3a10.9 10.9 0 0 1-3.14 1.53 4.48 4.48 0 0 0-7.86 3v1A10.66 10.66 0 0 1 3 4s-4 9 5 13a11.64 11.64 0 0 1-7 2c9 5 20 0 20-11.5a4.5 4.5 0 0 0-.08-.83A7.72 7.72 0 0 0 23 3z"/>
              </svg>
            </a>
          </div>
        </div>
        <div class="footer-section">
          <h3>Quick Links</h3>
          <div class="footer-links">
            <a href="/#about">About Us</a>
            <a href="/#plans">Plans</a>
            <a href="/#faq">FAQ</a>
          </div>
        </div>
        <div class="footer-section">
          <h3>Contact Info</h3>
          <div class="contact-info">
            <a href="mailto:support@listenwell.com" class="email">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/>
                <polyline points="22,6 12,13 2,6"/>
              </svg>
              oliwertwist9@gmail.com
            </a>
            <a href="tel:+1234567890" class="phone">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>
              </svg>
              +1234567890
            </a>
          </div>
        </div>
      </div>
      <div class="footer-bottom">
        <p>&copy; 2024 ListenWell. All rights reserved.</p>
      </div>
    </footer>
  </Route>

  <Route path="/plan/:slug" let:params>
    <Plan {params} />
  </Route>
</Router>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Space Grotesk', -apple-system, BlinkMacSystemFont, sans-serif;
    line-height: 1.6;
    color: var(--text);
    background: var(--bg);
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px 40px;
    border-bottom: 2px solid var(--yellow);
    margin-bottom: 40px;
    position: relative;
    background: #fff8e1;
  }

  .nav-links {
    display: flex;
    gap: 20px;
  }

  .nav-links a {
    text-decoration: none;
    color: var(--text);
    font-weight: 500;
  }

  .nav-links a:hover {
    color: var(--green);
  }

  .menu-toggle {
    display: none;
    background: none;
    border: none;
    cursor: pointer;
    padding: 5px;
  }

  .logo {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .logo span {
    font-weight: 600;
    font-size: 24px;
    color: var(--text);
    letter-spacing: -0.5px;
  }

  header {
    text-align: center;
    margin: 60px 0;
    padding: 0 20px;
  }

  .header-stats {
    display: flex;
    justify-content: center;
    gap: 40px;
    margin-top: 40px;
    flex-wrap: wrap;
  }

  .stat {
    text-align: center;
    min-width: 200px;
    padding: 20px;
    border-radius: 10px;
  }

  .stat:nth-child(1) {
    background: #ffebee;
  }

  .stat:nth-child(2) {
    background: #fff8e1;
  }

  .stat:nth-child(3) {
    background: #e8f5e9;
  }

  .stat .number {
    font-size: 32px;
    font-weight: 600;
  }

  .stat:nth-child(1) .number {
    color: #d32f2f;
  }

  .stat:nth-child(2) .number {
    color: #f57c00;
  }

  .stat:nth-child(3) .number {
    color: #388e3c;
  }

  .plans {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 30px;
    margin: 40px 0;
    padding: 0 20px;
    max-width: 1000px;
    margin: 40px auto;
  }

  .plan {
    border: 2px solid var(--text);
    padding: 30px;
    background: var(--bg);
    border-radius: 10px;
    transition: transform 0.3s ease;
    display: flex;
    flex-direction: column;
  }

  .plan:hover {
    transform: translateY(-5px);
  }

  .plan.popular {
    border-color: var(--green);
    box-shadow: 0 0 20px rgba(34, 197, 94, 0.1);
  }

  .plan-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .plan-header h2 {
    font-size: 24px;
  }

  .price {
    font-size: 32px;
    margin: 20px 0;
    font-weight: 600;
  }

  .period {
    font-size: 16px;
    font-weight: normal;
    opacity: 0.8;
  }

  .description {
    margin: 15px 0;
    color: var(--text);
    opacity: 0.8;
  }

  .plan ul {
    flex-grow: 1;
    margin-bottom: 20px;
  }

  .plan a.primary {
    display: inline-block;
    padding: 12px 24px;
    background: #ffebee;
    color: #d32f2f;
    text-decoration: none;
    border-radius: 5px;
    text-align: center;
    transition: background 0.3s ease;
  }

  .plan.popular a.primary {
    background: #ffebee;
    color: #d32f2f;
  }

  .plan a.primary:hover {
    background: #ffcdd2;
  }

  .video-section {
    padding: 80px 0;
    background: #fff8e1;
  }

  .video-container {
    max-width: 800px;
    margin: 0 auto;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 0 20px;
  }

  .video-container iframe {
    width: 100%;
    height: 450px;
    border: none;
  }

  .video-content {
    max-width: 800px;
    margin: 40px auto 0;
    padding: 0 20px;
  }

  .video-content h2 {
    margin-bottom: 20px;
  }

  .video-content ul {
    list-style: none;
    padding: 0;
    margin-top: 20px;
  }

  .video-content li {
    display: flex;
    align-items: center;
    gap: 10px;
    margin: 10px 0;
  }

  .video-content li::before {
    content: "•";
    color: var(--text);
    font-size: 20px;
  }

  .faq-section {
    padding: 80px 0;
    background: #ffebee;
    margin-bottom: 0;
  }

  .faq-section h2 {
    text-align: center;
    margin-bottom: 40px;
    color: var(--text);
  }

  .faq-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 20px;
  }

  .faq-item {
    margin-bottom: 20px;
    border: 1px solid var(--border);
    border-radius: 8px;
    overflow: hidden;
    background: white;
  }

  .faq-question {
    width: 100%;
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #fff8e1;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .faq-question:hover {
    background: #ffebee;
  }

  .faq-question span {
    font-size: 18px;
    font-weight: 500;
    color: var(--text);
  }

  .faq-question .icon {
    transition: transform 0.3s ease;
  }

  .faq-question .icon.rotate {
    transform: rotate(180deg);
  }

  .faq-answer {
    padding: 0 20px;
    max-height: 0;
    overflow: hidden;
    transition: all 0.3s ease;
  }

  .faq-item.open .faq-answer {
    padding: 20px;
    max-height: 500px;
  }

  .faq-answer p {
    color: var(--text-light);
    line-height: 1.6;
  }

  footer {
    background: #e8f5e9;
    color: #333;
    padding: 60px 0 20px;
    margin-top: 0;
  }

  .footer-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 40px;
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
  }

  .footer-section {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .footer-section h3 {
    color: #333;
    font-size: 18px;
    margin-bottom: 15px;
  }

  .footer-links {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .footer-links a {
    color: #333;
    text-decoration: none;
    transition: color 0.3s ease;
  }

  .footer-links a:hover {
    color: #388e3c;
  }

  .social-links {
    display: flex;
    gap: 20px;
    margin-top: 20px;
  }

  .social-links a {
    color: #333;
    transition: all 0.3s ease;
  }

  .social-links a:hover {
    transform: translateY(-3px);
  }

  .social-links .peerlist:hover {
    color: #00A67E;
  }

  .social-links .github:hover {
    color: #333;
  }

  .social-links .twitter:hover {
    color: #1DA1F2;
  }

  .contact-info {
    display: flex;
    flex-direction: column;
    gap: 15px;
  }

  .contact-info a {
    display: flex;
    align-items: center;
    gap: 12px;
    color: #333;
    text-decoration: none;
    transition: all 0.3s ease;
  }

  .contact-info a:hover {
    color: #388e3c;
    transform: translateX(5px);
  }

  .footer-bottom {
    text-align: center;
    margin-top: 40px;
    padding-top: 20px;
    border-top: 1px solid rgba(0, 0, 0, 0.1);
  }

  @media (max-width: 768px) {
    nav {
      padding: 20px;
    }

    .nav-links {
      display: none;
      position: absolute;
      top: 100%;
      left: 0;
      right: 0;
      background: white;
      padding: 20px;
      flex-direction: column;
      gap: 15px;
      border-bottom: 2px solid var(--yellow);
    }

    .nav-links.active {
      display: flex;
    }

    .menu-toggle {
      display: block;
    }

    .video-container iframe {
      height: 300px;
    }

    .header-stats {
      flex-direction: column;
      gap: 20px;
    }

    .plans {
      grid-template-columns: 1fr;
      padding: 0 20px;
    }

    .footer-content {
      grid-template-columns: 1fr;
      text-align: center;
      gap: 40px;
    }

    .footer-section {
      align-items: center;
    }

    .footer-section p {
      max-width: none;
    }

    .social-links {
      justify-content: center;
    }

    .contact-info a {
      justify-content: center;
    }

    .form {
      padding: 20px;
    }

    .radio-group {
      flex-direction: column;
    }
  }

  @media (max-width: 480px) {
    .video-container iframe {
      height: 200px;
    }

    .plan {
      padding: 20px;
    }

    .price {
      font-size: 24px;
    }

    .faq-question span {
      font-size: 16px;
    }
  }

  .form {
    max-width: 600px;
    margin: 40px auto;
    padding: 30px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .form h2 {
    margin-bottom: 30px;
    color: var(--text);
  }

  .form-group {
    margin-bottom: 20px;
  }

  .form-group label {
    display: block;
    margin-bottom: 8px;
    color: #333;
    font-weight: 500;
  }

  .form-group input,
  .form-group textarea {
    width: 100%;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 6px;
    font-size: 16px;
    transition: border-color 0.3s ease;
  }

  .form-group input:focus,
  .form-group textarea:focus {
    outline: none;
    border-color: #388e3c;
  }

  .form-group textarea {
    resize: vertical;
    min-height: 100px;
  }

  .radio-group {
    display: flex;
    gap: 20px;
    margin-top: 10px;
  }

  .radio-group label {
    display: flex;
    align-items: center;
    gap: 8px;
    cursor: pointer;
  }

  .radio-group input[type="radio"] {
    width: auto;
    margin: 0;
  }

  .buttons {
    display: flex;
    gap: 10px;
    margin-top: 30px;
  }

  .error {
    color: var(--red);
    margin: 10px 0;
    padding: 10px;
    background: rgba(255, 118, 118, 0.1);
    border-radius: 5px;
  }

  .success {
    text-align: center;
    padding: 40px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    max-width: 600px;
    margin: 40px auto;
  }

  .success h2 {
    color: #388e3c;
    margin-bottom: 20px;
  }

  .success p {
    color: #333;
    margin-bottom: 10px;
    font-size: 18px;
  }

  .success .icon {
    width: 60px;
    height: 60px;
    margin-bottom: 20px;
    color: #388e3c;
  }

  .success button {
    margin-top: 20px;
  }

  @media (max-width: 768px) {
    .form {
      padding: 20px;
      margin: 20px;
    }

    .radio-group {
      flex-direction: column;
      gap: 10px;
    }

    .buttons {
      flex-direction: column;
    }

    .buttons button {
      width: 100%;
    }
  }

  .back-link {
    margin-bottom: 20px;
  }

  .back-link button {
    display: flex;
    align-items: center;
    gap: 8px;
    background: none;
    border: none;
    color: var(--text);
    font-size: 16px;
    cursor: pointer;
    padding: 8px 0;
  }

  .back-link button:hover {
    color: var(--green);
  }

  .back-link svg {
    width: 20px;
    height: 20px;
  }

  .plans-section {
    text-align: center;
    padding: 40px 0;
    background: #e8f5e9;
  }

  .plans-section h2 {
    margin-bottom: 30px;
    color: var(--text);
  }

  .faq-section {
    padding: 80px 0;
    background: #ffebee;
  }

  .faq-section h2 {
    text-align: center;
    margin-bottom: 40px;
    color: var(--text);
  }

  .faq-item {
    margin-bottom: 20px;
    border: 1px solid var(--border);
    border-radius: 8px;
    overflow: hidden;
    background: white;
  }

  .faq-question {
    width: 100%;
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #fff8e1;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .faq-question:hover {
    background: #ffebee;
  }
</style>
