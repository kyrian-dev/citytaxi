<script setup>
import { computed, ref, watch, onMounted } from 'vue'

const language = ref('en')

const pickup = ref('')
const dropoff = ref('')
const date = ref('')
const time = ref('')
const passengers = ref('1')

const distance = ref(5)

const estimatedFare = computed(() => {
  const km = Number(distance.value)

  if (!Number.isFinite(km) || km < 1 || km > 500) {
    return null
  }

  // Illustrative demo formula: €4 base + €2 per kilometre.
  return new Intl.NumberFormat(
    language.value === 'en' ? 'en-IE' : 'de-DE',
    { style: 'currency', currency: 'EUR' }
  ).format(4 + km * 2)
})

const bookingConfirmed = ref(false)

function requestBooking() {
  bookingConfirmed.value = false

  const selectedDateTime = new Date(`${date.value}T${time.value}`)

  if (
    Number.isNaN(selectedDateTime.getTime()) ||
    selectedDateTime <= new Date()
  ) {
    alert(
      language.value === 'en'
        ? 'Please choose a future date and time.'
        : 'Bitte wähle ein Datum und eine Uhrzeit in der Zukunft.'
    )
    return
  }

  bookingConfirmed.value = true
}

watch(
  [pickup, dropoff, date, time, passengers, distance],
  () => {
    bookingConfirmed.value = false
  }
)

const bookings = ref([])
const storageUnavailable = ref(false)

onMounted(() => {
  try {
    const saved = JSON.parse(
      localStorage.getItem('citytaxi-demo-bookings') || '[]'
    )

    if (Array.isArray(saved)) {
      bookings.value = saved
        .filter(
          (item) =>
            item &&
            typeof item.id === 'string' &&
            typeof item.pickup === 'string' &&
            typeof item.dropoff === 'string'
        )
        .slice(0, 10)
    }
  } catch {
    storageUnavailable.value = true
  }
})

function persistBookings() {
  try {
    localStorage.setItem(
      'citytaxi-demo-bookings',
      JSON.stringify(bookings.value)
    )
    storageUnavailable.value = false
  } catch {
    storageUnavailable.value = true
  }
}

function saveDemoBooking() {
  bookings.value = [
    {
      id: crypto.randomUUID(),
      pickup: pickup.value.trim(),
      dropoff: dropoff.value.trim(),
      date: date.value,
      time: time.value,
      passengers: passengers.value,
      fare: estimatedFare.value,
    },
    ...bookings.value,
  ].slice(0, 10)

  persistBookings()
}

function removeBooking(id) {
  bookings.value = bookings.value.filter((booking) => booking.id !== id)
  persistBookings()
}
</script>

<template>
  <main class="app">
    <header class="header">
      <a class="brand" href="#" aria-label="CityTaxi">
  <img class="brand-logo" src="/logo.svg" alt="" width="48" height="48" />
  <span>City<span class="brand-accent">Taxi</span></span>
   </a>

      <button
        type="button"
        @click="language = language === 'en' ? 'de' : 'en'"
      >
        {{ language === 'en' ? 'Deutsch' : 'English' }}
      </button>
    </header>

    <section class="hero">
      <div class="hero-layout">
        <div class="hero-content">

      <p class="eyebrow">
        {{ language === 'en' ? 'YOUR CITY. YOUR RIDE.' : 'DEINE STADT. DEINE FAHRT.' }}
      </p>

      <h1>
        {{
          language === 'en'
            ? 'Your next ride starts here.'
            : 'Deine nächste Fahrt beginnt hier.'
        }}
      </h1>

      <p>
        {{
          language === 'en'
            ? 'Plan your trip in a few simple steps.'
            : 'Plane deine Fahrt in wenigen einfachen Schritten.'
        }}
      </p>

      <a class="hero-button" href="#booking">
     {{ language === 'en' ? 'Book a ride' : 'Fahrt buchen' }}
     <span aria-hidden="true">↗</span>
     </a>

      <p class="demo-note">
        {{
          language === 'en'
            ? 'Portfolio demo — no real taxi bookings are sent.'
            : 'Portfolio-Demo — es werden keine echten Taxibuchungen versendet.'
        }}
      </p>
          </div>

    <div class="hero-photo">
      <img
        src="/images/taxi.jpg"
        :alt="language === 'en' ? 'Taxi in the city' : 'Taxi in der Stadt'"
        width="1100"
        height="600"
        fetchpriority="high"
      />

      <span class="photo-badge">
        {{ language === 'en' ? 'Your city, Your ride.' : 'Deine Stadt. Deine Fahrt.' }}
      </span>
    </div>
    </div>
    </section>

    <section
         id="booking"
         class="booking-card"
      :aria-label="language === 'en' ? 'Plan your ride' : 'Fahrt planen'"
    >
      <h2>
        {{ language === 'en' ? 'Plan your ride' : 'Fahrt planen' }}
      </h2>

      <form @submit.prevent="requestBooking">
        <div class="booking-grid">
          <label>
            {{ language === 'en' ? 'Pickup address' : 'Abholadresse' }}
            <input
              v-model="pickup"
              type="text"
              :placeholder="language === 'en' ? 'Street, number, city' : 'Straße, Hausnummer, Stadt'"
              required
            />
          </label>

          <label>
            {{ language === 'en' ? 'Destination' : 'Zieladresse' }}
            <input
              v-model="dropoff"
              type="text"
              :placeholder="language === 'en' ? 'Where are you going?' : 'Wohin möchtest du fahren?'"
              required
            />
          </label>

          <label>
            {{ language === 'en' ? 'Date' : 'Datum' }}
            <input v-model="date" type="date" required />
          </label>

          <label>
            {{ language === 'en' ? 'Time' : 'Uhrzeit' }}
            <input v-model="time" type="time" required />
          </label>

          <label>
            {{ language === 'en' ? 'Passengers' : 'Fahrgäste' }}
            <select v-model="passengers">
              <option
                v-for="count in 6"
                :key="count"
                :value="String(count)"
              >
                {{ count }}
              </option>
            </select>
          </label>

          <label>
            {{
              language === 'en'
                ? 'Estimated distance (km)'
                : 'Geschätzte Entfernung (km)'
            }}
            <input
              v-model.number="distance"
              type="number"
              min="1"
              max="500"
              step="1"
              required
            />
          </label>
        </div>

        <div class="fare-estimate" aria-live="polite">
          <h3>
            {{ language === 'en' ? 'Illustrative fare' : 'Beispielpreis' }}
          </h3>

          <strong>{{ estimatedFare ?? '—' }}</strong>

          <p>
            {{
              language === 'en'
                ? 'Demo calculation only: €4 base fare + €2/km. Distance is entered manually. This is not a real quote.'
                : 'Nur eine Beispielrechnung: 4 € Grundpreis + 2 €/km. Die Entfernung wird manuell eingegeben. Dies ist kein echtes Preisangebot.'
            }}
          </p>
        </div>

        <button class="booking-button" type="submit">
          {{
            language === 'en'
              ? 'Request a ride — demo'
              : 'Fahrt anfragen — Demo'
          }}
        </button>
      </form>

      <div v-if="bookingConfirmed" class="confirmation" role="status">
        <h3>
          {{
            language === 'en'
              ? 'Demo request completed!'
              : 'Demo-Anfrage abgeschlossen!'
          }}
        </h3>

        <p>
          {{
            language === 'en'
              ? 'You have completed the demo booking flow. No request was sent and no taxi has been reserved.'
              : 'Du hast den Demo-Buchungsablauf abgeschlossen. Es wurde keine Anfrage versendet und kein Taxi reserviert.'
          }}
        </p>
        <dl class="trip-summary">
        <dt>{{ language === 'en' ? 'Pickup' : 'Abholung' }}</dt>
        <dd>{{ pickup }}</dd>

       <dt>{{ language === 'en' ? 'Destination' : 'Ziel' }}</dt>
       <dd>{{ dropoff }}</dd>

      <dt>{{ language === 'en' ? 'Date / time' : 'Datum / Uhrzeit' }}</dt>
      <dd>{{ date }} · {{ time }}</dd>

      <dt>{{ language === 'en' ? 'Passengers' : 'Fahrgäste' }}</dt>
      <dd>{{ passengers }}</dd>

      <dt>{{ language === 'en' ? 'Illustrative fare' : 'Beispielpreis' }}</dt>
      <dd>{{ estimatedFare }}</dd>
     </dl>
      </div>
    </section>
    <section class="contact-card">
    <h2>{{ language === 'en' ? 'Prefer to talk?' : 'Lieber persönlich?' }}</h2>

   <p>
    {{
      language === 'en'
        ? 'WhatsApp and phone booking will be available when a taxi operator connects their contact details.'
        : 'Buchungen per WhatsApp und Telefon sind verfügbar, sobald ein Taxiunternehmen seine Kontaktdaten hinterlegt.'
    }}
   </p>

   <div class="contact-actions">
    <button type="button" disabled>
      {{ language === 'en' ? 'WhatsApp — coming soon' : 'WhatsApp — demnächst' }}
    </button>

    <button type="button" disabled>
      {{ language === 'en' ? 'Call — coming soon' : 'Anrufen — demnächst' }}
    </button>
   </div>

   <small>
    {{ language === 'en' ? 'Portfolio demo · No operator connected' : 'Portfolio-Demo · Kein Taxiunternehmen verbunden' }}
   </small>
</section>

  <section class="dashboard">
  <h2>
    {{ language === 'en' ? 'Your demo bookings' : 'Deine Demo-Buchungen' }}
  </h2>

  <p class="dashboard-note">
    {{
      language === 'en'
        ? 'Last 10 requests, stored only in this browser. No taxi is reserved. Use fictional details.'
        : 'Die letzten 10 Anfragen, nur in diesem Browser gespeichert. Es wird kein Taxi reserviert. Bitte verwende erfundene Angaben.'
    }}
  </p>

  <p v-if="storageUnavailable" role="status">
    {{
      language === 'en'
        ? 'Browser storage is unavailable. Changes may be lost when you reload.'
        : 'Der Browserspeicher ist nicht verfügbar. Änderungen können beim Neuladen verloren gehen.'
    }}
  </p>

  <p v-if="bookings.length === 0">
    {{ language === 'en' ? 'No demo bookings yet.' : 'Noch keine Demo-Buchungen.' }}
  </p>

  <ul v-else class="booking-list">
    <li v-for="booking in bookings" :key="booking.id" class="saved-booking">
      <span class="demo-badge">
        {{ language === 'en' ? 'Demo request' : 'Demo-Anfrage' }}
      </span>

      <h3>{{ booking.pickup }} → {{ booking.dropoff }}</h3>

      <p>{{ booking.date }} · {{ booking.time }}</p>

      <p>
        {{ language === 'en' ? 'Passengers' : 'Fahrgäste' }}:
        {{ booking.passengers }}
      </p>

      <p>
        {{ language === 'en' ? 'Illustrative fare' : 'Beispielpreis' }}:
        <strong>{{ booking.fare }}</strong>
      </p>

      <button
        class="remove-button"
        type="button"
        @click="removeBooking(booking.id)"
      >
        {{ language === 'en' ? 'Delete saved demo' : 'Gespeicherte Demo löschen' }}
      </button>
    </li>
  </ul>
</section>
<footer class="footer">
  <a class="brand" href="#">CityTaxi</a>

  <p>
    {{
      language === 'en'
        ? 'A portfolio project built with Vue 3, JavaScript, Vite and CSS.'
        : 'Ein Portfolio-Projekt mit Vue 3, JavaScript, Vite und CSS.'
    }}
  </p>

  <small>
    {{
      language === 'en'
        ? 'Demo only. Live maps, real fares, dispatch and payments require external integrations and a back end.'
        : 'Nur eine Demo. Live-Karten, echte Fahrpreise, Fahrtenvermittlung und Zahlungen benötigen externe Anbindungen und ein Backend.'
    }}
  </small>
</footer>
  </main>
</template>