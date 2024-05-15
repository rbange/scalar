<script lang="ts">
  import { onMount } from 'svelte'
  import type { ReferenceConfiguration } from '@scalar/api-reference'
  import { svelteThemeCss } from './theme'

  export let configuration: ReferenceConfiguration
  let configString = ''
  let contentString = ''

  onMount(() => {
    initializeApiReference().catch(error => {
      console.error('Failed to initialize API reference:', error)
    })
  })

  const validateSpec = (spec) => {
    if (!spec?.content && !spec?.url) {
      throw new Error(
        '[@scalar/svelte-api-reference] You didn’t provide a spec.content or spec.url. Please provide one of these options.',
      )
    }
  }

  const applyDefaultCss = (config) => {
    if (!config?.customCss && !config?.theme) {
      config.customCss = svelteThemeCss
    }
  }

  const updateApiScript = (apiScript) => {
    apiScript.setAttribute('data-configuration', configString)
    apiScript.textContent = contentString
  }

  const sanitizeConfigString = (config) => {
    return JSON.stringify(config ?? {})
      .split('"')
      .join('&quot;')
  }

  const loadCdnScript = () => {
    const cdnScript = document.createElement('script')
    cdnScript.src = "https://cdn.jsdelivr.net/npm/@scalar/api-reference"
    document.body.appendChild(cdnScript)
  }

  const initializeApiReference = async () => {
    const { spec, ...config } = configuration
    const apiScript = document.getElementById('api-reference')

    validateSpec(spec)

    contentString = await getContentString(spec)
    applyDefaultCss(config)

    configString = sanitizeConfigString(config)

    if (apiScript) {
      updateApiScript(apiScript)
      loadCdnScript()
    }
  }

  const getContentString = async (spec) => {
    if (spec?.content) {
      return JSON.stringify(
        typeof spec.content === 'function' ? spec.content() : spec.content
      )
    } else if (spec?.url) {
      const response = await fetch(spec.url)
      const data = await response.json()
      return JSON.stringify(data)
    }
  }
</script>

<svelte:head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scalar API Reference</title>
</svelte:head>

<div>
  <script id="api-reference" type="application/json"></script>
</div>
