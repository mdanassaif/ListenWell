<script>
  import emailjs from "@emailjs/browser";
  import { plans } from "../data.js";
  import { goto } from '$app/navigation';

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
    goto(`/plan/${plan.title.toLowerCase().replace(/\s+/g, '-')}`, { replaceState: true });
  }

  async function handleSubmit() {
    loading = true;
    errorMessage = "";

    try {
      // Validate required fields
      if (!name || !email || !selectedDate || !selectedTime || !problem) {
        throw new Error("Please fill in all required fields");
      }

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
        session_duration: selectedPlan.features[0].split('-')[0].trim(),
        features_list: selectedPlan.features.map(feature => `<li>${feature}</li>`).join(''),
        session_details: `
          <div class="session-info">
            <p><strong>Session Type:</strong> ${selectedPlan.type}</p>
            <p><strong>Duration:</strong> ${selectedPlan.features[0].split('-')[0].trim()}</p>
            <p><strong>Date:</strong> ${selectedDate}</p>
            <p><strong>Time:</strong> ${selectedTime}</p>
            <p><strong>Contact Method:</strong> ${preferredContact}</p>
          </div>
        `
      };

      // Initialize EmailJS
      emailjs.init(EMAIL_PUBLIC_KEY);

      const response = await emailjs.send(
        EMAIL_SERVICE_ID,
        EMAIL_TEMPLATE_ID,
        templateParams
      );

      if (response.status === 200) {
        submitted = true;
        showBookingForm = false;
      } else {
        throw new Error("Failed to send email. Please try again.");
      }
    } catch (error) {
      console.error("EmailJS Error:", error);
      errorMessage = error.message || "Sorry, something went wrong. Please try again.";
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
    goto('/#plans', { replaceState: true });
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
</script>

<main>
  <nav>
    <div class="logo">
      <svg xmlns="http://www.w3.org/2000/svg" width="35" height="35" viewBox="0 0 24 24"><path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 21c.757.667 1.424 1 2 1c2 0 3-1 3-3c0-1.333.667-2.667 2-4c1.267-1.267 2-3.067 2-5a7 7 0 0 0-14 0m11 0a4 4 0 1 0-8 0M3 20l5-6l1 4l5-6"/></svg>
      <span>ListenWell</span>
    </div>
    <div class="nav-links">
      <a href="#about">About</a>
      <a href="#plans">Plans</a>
      <a href="#faq">FAQ</a>
      <a href="#contact">Contact</a>
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
        <li>
          <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M20 6L9 17l-5-5"/>
          </svg>
          Professional and trained listeners
        </li>
        <li>
          <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M20 6L9 17l-5-5"/>
          </svg>
          100% confidential sessions
        </li>
        <li>
          <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M20 6L9 17l-5-5"/>
          </svg>
          Flexible scheduling options
        </li>
        <li>
          <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M20 6L9 17l-5-5"/>
          </svg>
          Multiple support formats
        </li>
      </ul>
    </div>
  </section>

  {#if !submitted}
    {#if !showBookingForm}
      <div class="plans" id="plans">
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
            <div class="price">{plan.price} <span class="period">{plan.period}</span></div>
            <p class="description">{plan.description}</p>
            <ul>
              {#each plan.features as feature}
                <li>
                  <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M20 6L9 17l-5-5"/>
                  </svg>
                  {feature}
                </li>
              {/each}
            </ul>
            <button class="primary" on:click={() => selectPlan(plan)}>Choose Plan</button>
          </div>
        {/each}
      </div>
    {/if}
  {/if}

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

  {#if !submitted}
    {#if !showBookingForm}
      <div class="plans" id="plans">
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
            <div class="price">{plan.price} <span class="period">{plan.period}</span></div>
            <p class="description">{plan.description}</p>
            <ul>
              {#each plan.features as feature}
                <li>
                  <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M20 6L9 17l-5-5"/>
                  </svg>
                  {feature}
                </li>
              {/each}
            </ul>
            <button class="primary" on:click={() => selectPlan(plan)}>Choose Plan</button>
          </div>
        {/each}
      </div>
    {:else}
      <div class="form">
        <div class="back-link">
          <button on:click={() => {
            showBookingForm = false;
            selectedPlan = null;
            goto('/#plans', { replaceState: true });
          }}>
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M19 12H5M12 19l-7-7 7-7"/>
            </svg>
            Back to Plans
          </button>
        </div>
        <h2>Book Your {selectedPlan.title} Session</h2>
        <form on:submit|preventDefault={handleSubmit}>
          <div class="form-group">
            <label for="name">Name</label>
            <input type="text" id="name" bind:value={name} required />
          </div>
          <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" bind:value={email} required />
          </div>
          <div class="form-group">
            <label for="date">Preferred Date</label>
            <input type="date" id="date" bind:value={selectedDate} required />
          </div>
          <div class="form-group">
            <label for="time">Preferred Time</label>
            <input type="time" id="time" bind:value={selectedTime} required />
          </div>
          <div class="form-group">
            <label>Preferred Contact Method</label>
            <div class="radio-group">
              <label>
                <input type="radio" bind:group={preferredContact} value="email" />
                Email
              </label>
              <label>
                <input type="radio" bind:group={preferredContact} value="phone" />
                Phone
              </label>
              <label>
                <input type="radio" bind:group={preferredContact} value="whatsapp" />
                WhatsApp
              </label>
            </div>
          </div>
          <div class="form-group">
            <label for="problem">What would you like to talk about?</label>
            <textarea id="problem" bind:value={problem} required></textarea>
          </div>
          {#if errorMessage}
            <div class="error">{errorMessage}</div>
          {/if}
          <div class="buttons">
            <button type="submit" class="primary" disabled={loading}>
              {#if loading}
                <svg class="spinner" width="20" height="20" viewBox="0 0 24 24">
                  <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                </svg>
                Processing...
              {:else}
                Book Session
              {/if}
            </button>
          </div>
        </form>
      </div>
    {/if}
  {:else}
    <div class="success">
      <h2>Booking Confirmed! 🎉</h2>
      <p>Thanks {name}! We've sent you an email with session details.</p>
      <p>We'll be in touch shortly to confirm your {selectedPlan.title} session.</p>
      <button on:click={startOver}>Book Another Session</button>
    </div>
  {/if}

  <footer>
    <div class="footer-content">
      <div class="logo">
        <svg xmlns="http://www.w3.org/2000/svg" width="35" height="35" viewBox="0 0 24 24"><path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 21c.757.667 1.424 1 2 1c2 0 3-1 3-3c0-1.333.667-2.667 2-4c1.267-1.267 2-3.067 2-5a7 7 0 0 0-14 0m11 0a4 4 0 1 0-8 0M3 20l5-6l1 4l5-6"/></svg>
        <span>ListenWell</span>
      </div>
      <p class="copyright">&copy; 2024 ListenWell</p>
    </div>
  </footer>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Space Grotesk', -apple-system, BlinkMacSystemFont, sans-serif;
    line-height: 1.6;
    color: var(--text);
    background: var(--bg);
  }

  nav {
    display: flex;
    justify-content: center;
    padding: 20px;
    border-bottom: 2px solid var(--light-yellow);
    margin-bottom: 40px;
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

  .plans {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 30px;
    margin: 40px auto;
    width: 90%;
    max-width: 1200px;
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

  .plan:nth-child(1) {
    border-color: var(--light-yellow);
  }

  .plan:nth-child(2) {
    border-color: var(--light-green);
  }

  .plan:nth-child(1) button.primary {
    background: var(--light-yellow);
    color: var(--text);
  }

  .plan:nth-child(2) button.primary {
    background: var(--light-green);
    color: var(--text);
  }

  .plan-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .plan-header h2 {
    font-size: 24px;
    margin: 0;
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

  .plan button.primary {
    width: 100%;
    padding: 12px 24px;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .plan button.primary:hover {
    opacity: 0.9;
  }

  .stat:nth-child(1) .number {
    color: var(--light-red);
  }

  .stat:nth-child(2) .number {
    color: var(--light-yellow);
  }

  .stat:nth-child(3) .number {
    color: var(--light-green);
  }

  .video-content li:nth-child(1) .icon {
    color: var(--light-red);
  }

  .video-content li:nth-child(2) .icon {
    color: var(--light-yellow);
  }

  .video-content li:nth-child(3) .icon {
    color: var(--light-green);
  }

  .video-content li:nth-child(4) .icon {
    color: var(--light-red);
  }

  footer {
    background: var(--text);
    color: white;
    padding: 30px 20px;
    margin-top: 60px;
  }

  .footer-content {
    max-width: 400px;
    margin: 0 auto;
    text-align: center;
  }

  .footer-content .logo {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 12px;
    margin-bottom: 10px;
  }

  .footer-content .logo span {
    font-size: 24px;
    font-weight: 600;
    color: white;
  }

  .copyright {
    color: rgba(255, 255, 255, 0.6);
    font-size: 14px;
  }

  @media (max-width: 768px) {
    footer {
      padding: 20px;
    }

    .plans {
      grid-template-columns: 1fr;
      padding: 0 20px;
    }
  }
</style> 