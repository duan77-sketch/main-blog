<script>
  import { onMount } from 'svelte';

  // local-first loader then CDN fallback
  const LOCAL = '/vendor/live2d/autoload.js';
  const CDN = 'https://fastly.jsdelivr.net/npm/live2d-widgets@1.0.0/dist/autoload.js';

  function loadScript(src) {
    return new Promise((resolve, reject) => {
      // avoid duplicate
      if (document.querySelector(`script[src="${src}"]`)) return resolve();
      const s = document.createElement('script');
      s.src = src;
      s.defer = true;
      s.crossOrigin = 'anonymous';
      s.onload = () => resolve();
      s.onerror = () => reject(new Error('failed to load ' + src));
      document.head.appendChild(s);
    });
  }

  onMount(async () => {
    try {
      await loadScript(LOCAL);
      console.log('[HatsuneMiku] loaded local autoload.js');
    } catch (e) {
      console.warn('[HatsuneMiku] local autoload.js not found, trying CDN');
      try {
        await loadScript(CDN);
        console.log('[HatsuneMiku] loaded CDN autoload.js');
      } catch (err) {
        console.error('[HatsuneMiku] failed to load Live2D autoload.js', err);
        return;
      }
    }

    // ensure initWidget exists
    if (typeof window.initWidget !== 'function') {
      console.warn('[HatsuneMiku] initWidget not available after autoload.js');
      return;
    }

    // waifuPath should point to a waifu-tips.json that references the Hatsune Miku model files.
    // Place your model assets under `public/vendor/live2d/hatsune/` and update waifu-tips.json accordingly.
    const waifuConfig = {
      waifuPath: '/vendor/live2d/hatsune/waifu-tips.json',
      cubism2Path: '/vendor/live2d/live2d.min.js',
      cubism5Path: 'https://cubism.live2d.com/sdk-web/cubismcore/live2dcubismcore.min.js',
      tools: ['hitokoto', 'switch-model', 'photo', 'info', 'quit'],
      logLevel: 'warn',
      drag: false,
    };

    try {
      window.initWidget(waifuConfig);
      console.log('[HatsuneMiku] initWidget called with local hatsune config');
    } catch (err) {
      console.error('[HatsuneMiku] initWidget threw', err);
    }
  });
</script>

<style>
  :global(#waifu-toggle) { display: none; }
</style>

<div aria-hidden="true" style="display:none"></div>
