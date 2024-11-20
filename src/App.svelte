<script>
  import emailjs from "@emailjs/browser";
  import { plans } from "./data";

  let selectedPlan = null;
  let showBookingForm = false;
  let name = "";
  let email = "";
  let problem = "";
  let submitted = false;
  let loading = false;
  let errorMessage = "";

  const EMAIL_SERVICE_ID = import.meta.env.VITE_EMAIL_SERVICE_ID;
  const EMAIL_TEMPLATE_ID = import.meta.env.VITE_EMAIL_TEMPLATE_ID;
  const EMAIL_PUBLIC_KEY = import.meta.env.VITE_EMAIL_PUBLIC_KEY;

  function selectPlan(plan) {
    selectedPlan = plan;
    showBookingForm = true;
  }

  async function handleSubmit() {
    loading = true;
    errorMessage = "";

    try {
      const templateParams = {
        to_name: name,
        to_email: email,
        plan_name: selectedPlan.title,
        message: problem,
        price: selectedPlan.price,
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
  }
</script>

<main>
  <nav>
    <div class="logo">
      <img src="/listenwell.svg" alt="ListenWell Logo" />
      <span>ListenWell</span>
    </div>
  </nav>

  <header>
    <h1>Someone to Talk To</h1>
    <p>Professional listening service. No judgment, just support.</p>
  </header>

  {#if !submitted}
    {#if !showBookingForm}
      <div class="plans">
        {#each plans as plan}
          <div class="plan">
            <h2>{plan.title}</h2>
            <div class="price">{plan.price}</div>
            <ul>
              {#each plan.features as feature}
                <li>{feature}</li>
              {/each}
            </ul>
            <button on:click={() => selectPlan(plan)}> Choose Plan </button>
          </div>
        {/each}
      </div>
    {:else}
      <div class="form">
        <h2>Book Your Session</h2>
        <form on:submit|preventDefault={handleSubmit}>
          <div>
            <label for="name">Name</label>
            <input type="text" id="name" bind:value={name} required />
          </div>
          <div>
            <label for="email">Email</label>
            <input type="email" id="email" bind:value={email} required />
          </div>
          <div>
            <label for="problem">What would you like to talk about?</label>
            <textarea id="problem" bind:value={problem} required></textarea>
          </div>
          {#if errorMessage}
            <div class="error">{errorMessage}</div>
          {/if}
          <div class="buttons">
            <button type="submit">Book Session</button>
            <button type="button" on:click={() => (showBookingForm = false)}
              >Back</button
            >
          </div>
        </form>
      </div>
    {/if}
  {:else}
    <div class="success">
      <h2>Booking Confirmed</h2>
      <p>Thanks {name}! We've sent you an email with session details.</p>
      <button on:click={startOver}>Book Another Session</button>
    </div>
  {/if}
</main>
