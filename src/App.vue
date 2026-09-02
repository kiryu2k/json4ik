<script setup lang="ts">
  import { computed, ref, watch } from 'vue'

  const input = ref(`{
  "name": "json4ik",
  "version": "1.0.0",
  "features": [
    "format",
    "minify",
    "copy"
  ],
  "active": true
}`)

  const output = ref('')
  const error = ref('')
  const copied = ref(false)

  const hasOutput = computed(() => output.value.length > 0)

  const isValidJson = ref<boolean | null>(null)
  watch(input, () => {
    if (!input.value.trim()) {
      isValidJson.value = null
      return
    }

    try {
      JSON.parse(input.value)
      isValidJson.value = true
    } catch {
      isValidJson.value = false
    }
  })

  const parseJson = () => {
    error.value = ''

    try {
      return JSON.parse(input.value)
    } catch (err) {
      if (err instanceof SyntaxError) {
        error.value = `invalid JSON: ${err.message}`
      } else {
        error.value = 'invalid JSON'
      }

      return null
    }
  }

  const formatJson = () => {
    const parsed = parseJson()

    if (parsed === null) {
      output.value = ''
      return
    }

    output.value = JSON.stringify(parsed, null, 2)
  }

  const minifyJson = () => {
    const parsed = parseJson()

    if (parsed === null) {
      output.value = ''
      return
    }

    output.value = JSON.stringify(parsed)
  }

  const copyJson = async () => {
    if (!output.value) return

    try {
      await navigator.clipboard.writeText(output.value)

      copied.value = true

      setTimeout(() => {
        copied.value = false
      }, 1500)
    } catch {
      error.value = 'failed to copy JSON'
    }
  }

  const clearAll = () => {
    input.value = ''
    output.value = ''
    error.value = ''
    copied.value = false
  }
</script>

<template>
  <div class="app">
    <header class="header">
      <div class="brand">
        <div class="logo">{ }</div>

        <div>
          <h1>json4ik</h1>
          <p>format, minify and validate your JSON</p>
        </div>
      </div>

      <div class="header-actions">
        <a class="github-button" href="https://github.com/kiryu2k" target="_blank" rel="noopener noreferrer"
          aria-label="GitHub">
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <path fill="currentColor" d="M12 2C6.477 2 2 6.477 2 12c0 4.418 2.865 8.166 6.839 9.49.5.092.682-.217.682-.482
        0-.237-.009-.866-.014-1.7-2.782.604-3.369-1.34-3.369-1.34-.455-1.156-1.11-1.464-1.11-1.464
        -.908-.62.069-.608.069-.608 1.004.07 1.532 1.032 1.532 1.032.892 1.529 2.341 1.087
        2.91.831.091-.647.349-1.087.636-1.338-2.22-.253-4.555-1.11-4.555-4.943
        0-1.091.39-1.984 1.029-2.683-.103-.253-.446-1.27.098-2.647 0 0 .84-.269
        2.75 1.025A9.564 9.564 0 0 1 12 6.844a9.58 9.58 0 0 1 2.504.337
        c1.909-1.294 2.748-1.025 2.748-1.025.546 1.377.202 2.394.1 2.647
        .64.699 1.028 1.592 1.028 2.683 0 3.842-2.339 4.687-4.566 4.935
        .359.309.678.919.678 1.852 0 1.336-.012 2.415-.012 2.744
        0 .267.18.578.688.48A10.001 10.001 0 0 0 22 12C22 6.477 17.523 2 12 2Z" />
          </svg>
          my github
        </a>

        <button class="clear-button" @click="clearAll">
          clear
        </button>
      </div>
    </header>

    <main class="container">
      <section class="toolbar">
        <div class="actions">
          <button class="button primary" @click="formatJson">
            <span>↗</span>
            format
          </button>

          <button class="button" @click="minifyJson">
            <span>≡</span>
            minify
          </button>

          <button class="button" :disabled="!hasOutput" @click="copyJson">
            <span>{{ copied ? '✓' : '□' }}</span>
            {{ copied ? 'copied!' : 'copy' }}
          </button>
        </div>

        <div class="status" :class="{
          valid: isValidJson === true,
          error: isValidJson === false
        }">
          <span v-if="isValidJson === true">
            ● valid JSON
          </span>

          <span v-else-if="isValidJson === false">
            ● invalid JSON
          </span>

          <span v-else>
            ● ready
          </span>
        </div>
      </section>

      <div class="editors">
        <!-- INPUT -->
        <section class="editor-panel">
          <div class="panel-header">
            <div class="panel-title">
              <span class="dot input-dot"></span>
              input
            </div>

            <span class="panel-label">JSON</span>
          </div>

          <textarea v-model="input" class="editor" spellcheck="false" placeholder="Paste your JSON here..."
            aria-label="JSON input"></textarea>
        </section>

        <!-- OUTPUT -->
        <section class="editor-panel">
          <div class="panel-header">
            <div class="panel-title">
              <span class="dot output-dot"></span>
              output
            </div>

            <span class="panel-label">JSON</span>
          </div>

          <pre v-if="hasOutput" class="editor output-editor"><code>{{ output }}</code></pre>

          <div v-else class="empty-state">
            <div class="empty-icon">{ }</div>
            <h2>your formatted JSON will appear here</h2>
            <p>paste JSON on the left and click «format»</p>
          </div>
        </section>
      </div>

      <div v-if="error" class="error-box">
        <div class="error-icon">!</div>

        <div>
          <strong>invalid JSON</strong>
          <p>{{ error }}</p>
        </div>
      </div>

      <footer>
        <span>json4ik</span>
        <span>•</span>
        <span>client-side · your data stays in your browser</span>
      </footer>
    </main>
  </div>
</template>
