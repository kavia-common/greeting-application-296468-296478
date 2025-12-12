<script setup lang="ts">
import { onMounted, ref } from 'vue'

const greeting = ref<string>('')
const loading = ref<boolean>(false)
const error = ref<string | null>(null)

// Determine API base dynamically using current hostname and port 3001
const apiBase = `http://${window.location.hostname}:3001`

async function fetchGreeting() {
  loading.value = true
  error.value = null
  try {
    const res = await fetch(`${apiBase}/hello`, {
      method: 'GET',
      headers: {
        'Accept': 'text/plain',
      },
    })

    if (!res.ok) {
      // Try to get text for better error messages; fallback to status text
      let detail = ''
      try {
        detail = await res.text()
      } catch {
        detail = res.statusText
      }
      throw new Error(`Request failed (${res.status}): ${detail || 'Unknown error'}`)
    }

    const text = await res.text()
    greeting.value = text
  } catch (e: unknown) {
    const msg = e instanceof Error ? e.message : 'Unexpected error'
    error.value = msg
    greeting.value = ''
  } finally {
    loading.value = false
  }
}

function onRefresh() {
  fetchGreeting()
}

onMounted(() => {
  fetchGreeting()
})
</script>

<template>
  <main class="page">
    <section class="hero">
      <div class="card">
        <div class="card-accent" aria-hidden="true"></div>
        <header class="card-header">
          <h1 class="title">Welcome</h1>
          <p class="subtitle">Fetch a greeting from the backend service</p>
        </header>

        <div class="card-body">
          <div v-if="loading" class="state state-loading" role="status" aria-live="polite">
            <span class="spinner" aria-hidden="true"></span>
            <span>Loading greeting...</span>
          </div>

          <div v-else-if="error" class="state state-error" role="alert">
            <span class="error-dot" aria-hidden="true">!</span>
            <div>
              <strong>Failed to load greeting</strong>
              <div class="error-text">{{ error }}</div>
            </div>
          </div>

          <div v-else class="greeting">
            <p class="greeting-text">{{ greeting }}</p>
          </div>
        </div>

        <footer class="card-footer">
          <button class="btn btn-primary" type="button" @click="onRefresh" :disabled="loading">
            <span v-if="!loading">Refresh</span>
            <span v-else>Refreshing...</span>
          </button>
          <span class="hint">Backend: {{ apiBase }}/hello</span>
        </footer>
      </div>
    </section>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  background: #f9fafb; /* Ocean Professional background */
  padding: 2rem;
}

.hero {
  width: 100%;
  max-width: 720px;
}

.card {
  position: relative;
  background: #ffffff; /* Surface */
  color: #111827;      /* Text */
  border-radius: 16px;
  box-shadow:
    0 10px 15px -3px rgba(0, 0, 0, 0.08),
    0 4px 6px -2px rgba(0, 0, 0, 0.04);
  overflow: hidden;
  border: 1px solid rgba(17, 24, 39, 0.06);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow:
    0 15px 25px -5px rgba(0, 0, 0, 0.1),
    0 8px 10px -6px rgba(0, 0, 0, 0.08);
}

.card-accent {
  height: 6px;
  background: linear-gradient(90deg, rgba(37, 99, 235, 0.25), rgba(245, 158, 11, 0.25));
}

.card-header {
  padding: 1.25rem 1.5rem 0.25rem;
}

.title {
  font-size: 1.75rem;
  line-height: 1.2;
  margin: 0;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.subtitle {
  margin-top: 0.25rem;
  color: #4b5563;
  font-size: 0.95rem;
}

.card-body {
  padding: 1.5rem;
}

.greeting {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1.25rem;
  border-radius: 12px;
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.06), rgba(249, 250, 251, 1));
  border: 1px solid rgba(37, 99, 235, 0.12);
}

.greeting-text {
  font-size: 1.25rem;
  font-weight: 600;
  color: #111827;
  text-align: center;
}

.state {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  padding: 1rem 1.25rem;
  border-radius: 12px;
}

.state-loading {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid rgba(55, 65, 81, 0.15);
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(37, 99, 235, 0.25);
  border-top-color: #2563EB; /* primary */
  border-radius: 50%;
  display: inline-block;
  animation: spin 0.9s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.state-error {
  background: #fef2f2;
  color: #991b1b;
  border: 1px solid rgba(239, 68, 68, 0.3);
}

.error-dot {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #ef4444;
  color: white;
  font-weight: 700;
}

.error-text {
  color: #b91c1c;
  font-size: 0.9rem;
}

.card-footer {
  padding: 1rem 1.5rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.btn {
  appearance: none;
  border: none;
  cursor: pointer;
  border-radius: 10px;
  padding: 0.6rem 1rem;
  font-weight: 600;
  transition: background-color 0.2s ease, box-shadow 0.2s ease, transform 0.06s ease;
}

.btn-primary {
  background: #2563EB;
  color: #ffffff;
  box-shadow: 0 6px 10px -4px rgba(37, 99, 235, 0.45);
}

.btn-primary:hover {
  background: #1d4ed8;
  box-shadow: 0 10px 15px -6px rgba(29, 78, 216, 0.55);
}

.btn-primary:active {
  transform: translateY(1px);
  background: #1e40af;
}

.btn[disabled],
.btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.hint {
  margin-left: auto;
  font-size: 0.85rem;
  color: #6b7280;
}

@media (max-width: 600px) {
  .card-footer {
    flex-direction: column;
    align-items: stretch;
  }
  .hint {
    margin-left: 0;
    text-align: center;
  }
}
</style>
