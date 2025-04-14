<script>
  import { onMount } from 'svelte';
  import { plans } from '../data.js';
  import emailjs from '@emailjs/browser';

  export let params;
  let plan = null;
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

  onMount(() => {
    plan = plans.find(p => p.title.toLowerCase().replace(/\s+/g, '-') === params.slug);
  });

  async function handleSubmit(event) {
    event.preventDefault();
    loading = true;
    errorMessage = "";

    try {
      const templateParams = {
        to_name: name,
        to_email: email,
        plan_name: plan.title,
        plan_price: plan.title === 'Free Support' ? 'Free' : '$0.1/month',
        plan_features: plan.features.map(feature => `• ${feature}`).join('\n'),
        plan_description: plan.description,
        message: problem,
        booking_date: selectedDate,
        booking_time: selectedTime,
        preferred_contact: preferredContact,
        support_email: "support@listenwell.com",
        website_url: "https://listenwell.vercel.app",
        session_type: plan.type,
        session_duration: plan.features[0].split('-')[0].trim(),
        email_template: `
          <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; background: #f8f9fa; border-radius: 8px;">
            <div style="text-align: center; margin-bottom: 30px;">
              <h1 style="color: #388e3c; margin-bottom: 10px;">ListenWell</h1>
              <h2 style="color: #333; margin-bottom: 20px;">Your Session Booking Confirmation</h2>
            </div>

            <div style="background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
              <p style="color: #333; font-size: 16px; line-height: 1.6;">Hello {{to_name}},</p>
              
              <p style="color: #333; font-size: 16px; line-height: 1.6;">Thank you for choosing ListenWell! We're excited to support you on your journey to better mental well-being.</p>

              <div style="margin: 20px 0; padding: 20px; background: #e8f5e9; border-radius: 6px;">
                <h3 style="color: #388e3c; margin-bottom: 15px;">Your Session Details</h3>
                <p style="margin: 10px 0;"><strong>Plan:</strong> {{plan_name}}</p>
                <p style="margin: 10px 0;"><strong>Description:</strong> {{plan_description}}</p>
                <p style="margin: 10px 0;"><strong>Price:</strong> {{plan_price}}</p>
                <p style="margin: 10px 0;"><strong>Session Type:</strong> {{session_type}}</p>
                <p style="margin: 10px 0;"><strong>Duration:</strong> {{session_duration}}</p>
                <p style="margin: 10px 0;"><strong>Date:</strong> {{booking_date}}</p>
                <p style="margin: 10px 0;"><strong>Time:</strong> {{booking_time}}</p>
              </div>

              <div style="margin: 20px 0;">
                <h3 style="color: #388e3c; margin-bottom: 15px;">What's Included</h3>
                <div style="color: #333; font-size: 16px; line-height: 1.6;">
                  {{plan_features}}
                </div>
              </div>

              <div style="margin: 20px 0;">
                <h3 style="color: #388e3c; margin-bottom: 15px;">Your Message</h3>
                <p style="color: #333; font-size: 16px; line-height: 1.6; background: #f8f9fa; padding: 15px; border-radius: 6px;">{{message}}</p>
              </div>

              <p style="color: #333; font-size: 16px; line-height: 1.6;">We'll contact you at <strong>{{to_email}}</strong> to confirm your session details and provide next steps.</p>

              <p style="color: #333; font-size: 16px; line-height: 1.6;">If you have any questions, please don't hesitate to contact us at <a href="mailto:{{support_email}}" style="color: #388e3c; text-decoration: none;">{{support_email}}</a></p>

              <div style="margin-top: 30px; padding-top: 20px; border-top: 1px solid #e8f5e9;">
                <p style="color: #666; font-size: 14px;">Best regards,<br>The ListenWell Team</p>
                <p style="margin-top: 20px;">
                  <a href="{{website_url}}" style="display: inline-block; padding: 10px 20px; background: #388e3c; color: white; text-decoration: none; border-radius: 4px;">Visit Our Website</a>
                </p>
              </div>
            </div>
          </div>
        `
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
  }
</script>

<div class="plan-page">
  {#if plan}
    <div class="back-link">
      <a href="/#plans">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
        Back to Plans
      </a>
    </div>

    {#if !submitted}
      <div class="form-container">
        <div class="form-header">
          <h1>Book Your {plan.title} Session</h1>
          {#if plan.title !== 'Free Support'}
            <div class="price-tag">
              <span class="price">$0.1</span>
              <span class="period">/month</span>
            </div>
          {/if}
        </div>

        <form on:submit={handleSubmit} class="booking-form">
          {#if errorMessage}
            <div class="error">{errorMessage}</div>
          {/if}

          <div class="form-group">
            <label for="name">Full Name</label>
            <input type="text" id="name" bind:value={name} required placeholder="Enter your full name" />
          </div>

          <div class="form-group">
            <label for="email">Email Address</label>
            <input type="email" id="email" bind:value={email} required placeholder="Enter your email address" />
          </div>

          <div class="form-group">
            <label for="problem">What would you like to discuss?</label>
            <textarea id="problem" bind:value={problem} required placeholder="Tell us what's on your mind..."></textarea>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label for="date">Preferred Date</label>
              <input type="date" id="date" bind:value={selectedDate} required />
            </div>

            <div class="form-group">
              <label for="time">Preferred Time</label>
              <input type="time" id="time" bind:value={selectedTime} required />
            </div>
          </div>

          <div class="form-group">
            <label>Preferred Contact Method</label>
            <div class="radio-group">
              <label>
                <input type="radio" bind:group={preferredContact} value="email" />
                <span class="radio-label">Email</span>
              </label>
              <label>
                <input type="radio" bind:group={preferredContact} value="phone" />
                <span class="radio-label">Phone</span>
              </label>
            </div>
          </div>

          <button type="submit" disabled={loading}>
            {#if loading}
              <svg class="spinner" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M21 12a9 9 0 1 1-6.219-8.56"/>
              </svg>
              Processing...
            {:else}
              Book Now
            {/if}
          </button>
        </form>
      </div>
    {:else}
      <div class="success">
        <svg class="icon" width="60" height="60" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/>
          <polyline points="22 4 12 14.01 9 11.01"/>
        </svg>
        <h2>Thank You!</h2>
        <p>We've received your booking request for {plan.title}.</p>
        <p>Our team will review your request and contact you shortly to confirm your session.</p>
        <p>You'll receive a confirmation email with all the details.</p>
        <a href="/#plans" class="back-button">Back to Plans</a>
      </div>
    {/if}
  {:else}
    <div class="not-found">
      <h1>Plan Not Found</h1>
      <p>The plan you're looking for doesn't exist.</p>
      <a href="/#plans">View Available Plans</a>
    </div>
  {/if}
</div>

<style>
  .plan-page {
    max-width: 800px;
    margin: 0 auto;
    padding: 40px 20px;
  }

  .back-link {
    margin-bottom: 30px;
  }

  .back-link a {
    display: flex;
    align-items: center;
    gap: 8px;
    color: #333;
    text-decoration: none;
    font-weight: 500;
  }

  .back-link a:hover {
    color: #388e3c;
  }

  .form-container {
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }

  .form-header {
    background: #e8f5e9;
    padding: 30px;
    text-align: center;
  }

  .form-header h1 {
    font-size: 28px;
    margin: 0 0 15px 0;
    color: #333;
  }

  .price-tag {
    display: flex;
    align-items: baseline;
    justify-content: center;
    gap: 5px;
  }

  .price {
    font-size: 32px;
    font-weight: 600;
    color: #388e3c;
  }

  .period {
    font-size: 16px;
    color: #666;
  }

  .booking-form {
    padding: 30px;
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
    transition: all 0.3s ease;
  }

  .form-group input:focus,
  .form-group textarea:focus {
    outline: none;
    border-color: #388e3c;
    box-shadow: 0 0 0 2px rgba(56, 142, 60, 0.1);
  }

  .form-group textarea {
    resize: vertical;
    min-height: 100px;
  }

  .form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
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
    padding: 10px 15px;
    background: #e8f5e9;
    border-radius: 6px;
    transition: all 0.3s ease;
  }

  .radio-group label:hover {
    background: #c8e6c9;
  }

  .radio-group input[type="radio"] {
    width: auto;
    margin: 0;
  }

  .radio-label {
    font-weight: 500;
  }

  button[type="submit"] {
    width: 100%;
    padding: 15px;
    background: #e8f5e9;
    color: #388e3c;
    border: none;
    border-radius: 6px;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    margin-top: 20px;
  }

  button[type="submit"]:hover {
    background: #c8e6c9;
    color: #2e7d32;
  }

  button[type="submit"]:disabled {
    background: #f1f8e9;
    color: #a5d6a7;
    cursor: not-allowed;
  }

  .spinner {
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    100% {
      transform: rotate(360deg);
    }
  }

  .success {
    text-align: center;
    padding: 40px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .success h2 {
    color: #388e3c;
    margin-bottom: 20px;
    font-size: 32px;
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

  .back-button {
    display: inline-block;
    margin-top: 30px;
    padding: 12px 24px;
    background: #388e3c;
    color: white;
    text-decoration: none;
    border-radius: 6px;
    transition: background 0.3s ease;
  }

  .back-button:hover {
    background: #2e7d32;
  }

  .not-found {
    text-align: center;
    padding: 60px;
  }

  .not-found h1 {
    font-size: 36px;
    margin-bottom: 20px;
  }

  .not-found p {
    font-size: 18px;
    margin-bottom: 30px;
  }

  .not-found a {
    display: inline-block;
    padding: 12px 24px;
    background: #388e3c;
    color: white;
    text-decoration: none;
    border-radius: 6px;
    transition: background 0.3s ease;
  }

  .not-found a:hover {
    background: #2e7d32;
  }

  @media (max-width: 768px) {
    .form-row {
      grid-template-columns: 1fr;
    }

    .radio-group {
      flex-direction: column;
      gap: 10px;
    }

    .form-header h1 {
      font-size: 24px;
    }

    .price {
      font-size: 28px;
    }
  }
</style> 