<script>
  import { plans } from '../../data.js';
  import emailjs from '@emailjs/browser';

  const selectedPlan = plans.find(p => p.title === "Basic Support");
  let name = "";
  let email = "";
  let problem = "";
  let submitted = false;
  let loading = false;
  let errorMessage = "";
  let selectedDate = "";
  let selectedTime = "";
  let preferredContact = "email";
  
  const EMAIL_SERVICE_ID = import.meta.env.VITE_EMAIL_SERVICE_ID;
  const EMAIL_TEMPLATE_ID = import.meta.env.VITE_EMAIL_TEMPLATE_ID;
  const EMAIL_PUBLIC_KEY = import.meta.env.VITE_EMAIL_PUBLIC_KEY;

  async function handleSubmit() {
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
    } catch (error) {
      errorMessage = "Sorry, something went wrong. Please try again.";
      console.error(error);
    } finally {
      loading = false;
    }
  }

  function startOver() {
    name = "";
    email = "";
    problem = "";
    errorMessage = "";
    selectedDate = "";
    selectedTime = "";
    preferredContact = "email";
    submitted = false;
    window.location.hash = 'plans';
  }
</script>

<main>
  <div class="form-container">
    {#if !submitted}
      <div class="form">
        <div class="back-link">
          <button on:click={() => window.location.hash = 'plans'}>
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M19 12H5M12 19l-7-7 7-7"/>
            </svg>
            Back to Plans
          </button>
        </div>
        <h2>Book Your Basic Support Session</h2>
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
    {:else}
      <div class="success">
        <h2>Booking Confirmed! 🎉</h2>
        <p>Thanks {name}! We've sent you an email with session details.</p>
        <p>We'll be in touch shortly to confirm your Basic Support session.</p>
        <button on:click={startOver}>Book Another Session</button>
      </div>
    {/if}
  </div>
</main>

<style>
  .form-container {
    max-width: 600px;
    margin: 40px auto;
    padding: 0 20px;
  }

  .form {
    background: white;
    border-radius: 12px;
    padding: 30px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
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
    color: var(--text);
    font-weight: 500;
  }

  .form-group input,
  .form-group textarea {
    width: 100%;
    padding: 12px;
    border: 1px solid var(--border);
    border-radius: 6px;
    font-size: 16px;
    transition: border-color 0.3s ease;
  }

  .form-group input:focus,
  .form-group textarea:focus {
    outline: none;
    border-color: var(--green);
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

  button {
    padding: 12px 24px;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  button.primary {
    background: var(--green);
    color: white;
  }

  button.primary:hover {
    background: var(--green-dark);
  }

  button.primary:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }

  .spinner {
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  .success {
    text-align: center;
    padding: 40px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .success h2 {
    color: var(--green);
    margin-bottom: 20px;
  }

  .success p {
    color: var(--text-light);
    margin-bottom: 10px;
  }

  .success button {
    margin-top: 20px;
  }

  .error {
    color: var(--red);
    margin: 10px 0;
    padding: 10px;
    background: rgba(255, 118, 118, 0.1);
    border-radius: 5px;
  }

  @media (max-width: 768px) {
    .form {
      padding: 20px;
    }

    .radio-group {
      flex-direction: column;
    }

    .buttons {
      flex-direction: column;
    }

    button {
      width: 100%;
    }
  }
</style> 