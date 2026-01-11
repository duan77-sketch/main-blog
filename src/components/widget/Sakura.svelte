<script>
  import { onMount, onDestroy } from 'svelte';

  // style prop removed (not used)

  let overlayEl;
  let running = true;
  let idSeq = 0;
  const MAX_PETALS = 36;
  const SPAWN_INTERVAL = 420; // base spawn interval

  const layers = [
    { speed: 0.6, scale: 0.75, blur: 1, opacity: 0.8 }, // far
    { speed: 1.0, scale: 1.0, blur: 0, opacity: 0.95 }, // mid
    { speed: 1.6, scale: 1.35, blur: 1.5, opacity: 1.0 } // near
  ];

  function rand(min, max) {
    return Math.random() * (max - min) + min;
  }

  function createPetalEl(size, layer) {
    const wrapper = document.createElement('div');
    wrapper.className = 'petal';
    wrapper.style.position = 'absolute';
    wrapper.style.top = '-10vh';
    wrapper.style.pointerEvents = 'none';
    wrapper.style.willChange = 'transform, opacity';

    // Construct an inline SVG petal for controllable shape and color
    const svgNS = 'http://www.w3.org/2000/svg';
    const svg = document.createElementNS(svgNS, 'svg');
    svg.setAttribute('width', String(size));
    svg.setAttribute('height', String(size));
    svg.setAttribute('viewBox', '0 0 24 24');
    svg.setAttribute('fill', 'none');

    const path = document.createElementNS(svgNS, 'path');
    path.setAttribute('d', 'M12 1 C15 3 20 8 12 22 C4 8 9 3 12 1 Z');
    path.setAttribute('fill', 'url(#sakuraGradient)');
    path.setAttribute('opacity', String(0.98));

    const defs = document.createElementNS(svgNS, 'defs');
    const grad = document.createElementNS(svgNS, 'linearGradient');
    grad.setAttribute('id', 'sakuraGradient');
    grad.setAttribute('x1', '0');
    grad.setAttribute('x2', '1');
    grad.setAttribute('y1', '0');
    grad.setAttribute('y2', '1');

    const stop1 = document.createElementNS(svgNS, 'stop');
    stop1.setAttribute('offset', '0');
    stop1.setAttribute('stop-color', '#ffe3ef');
    stop1.setAttribute('stop-opacity', '1');

    const stop2 = document.createElementNS(svgNS, 'stop');
    stop2.setAttribute('offset', '1');
    stop2.setAttribute('stop-color', '#ffb6d5');
    stop2.setAttribute('stop-opacity', '1');

    grad.appendChild(stop1);
    grad.appendChild(stop2);
    defs.appendChild(grad);

    svg.appendChild(defs);
    svg.appendChild(path);

    wrapper.appendChild(svg);
    return wrapper;
  }

  function spawnPetal() {
    if (!running) return;
    if (!overlayEl) return;
    const id = idSeq++;

    // mobile heuristic: reduce/disable on small viewports
    if (window.innerWidth < 640) {
      if (Math.random() > 0.12) return; // heavy throttle on mobile
    }

    if (overlayEl.childElementCount >= MAX_PETALS) return;

    const startX = rand(-8, 108); // percent
    const layerIdx = Math.floor(rand(0, layers.length));
    const layer = layers[layerIdx];
    const size = rand(14, 28) * layer.scale; // px
    const duration = rand(6500, 14000) / layer.speed; // faster in near layer
    const sway = rand(40, 160) * layer.scale; // px horizontal amplitude
    const phase = rand(0, Math.PI * 2);
    const rotation = rand(-120, 120);
    const spin = rand(-360, 360);

    const el = createPetalEl(size, layer);
    el.style.left = startX + '%';
    el.style.opacity = String(layer.opacity);
    if (layer.blur > 0) {
      el.style.filter = `blur(${layer.blur}px) drop-shadow(0 6px 12px rgba(0,0,0,0.06))`;
    }

    overlayEl.appendChild(el);

    const petal = {
      id,
      el,
      startX,
      layerIdx,
      size,
      duration,
      sway,
      phase,
      rotation,
      spin,
      born: performance.now()
    };

    // Clean up after duration + buffer
    setTimeout(() => {
      try { el.remove(); } catch (e) {}
    }, duration + 220);

    // store on the DOM element for the animation loop
    el.dataset.petalId = String(id);
  }

  let rafId;
  function animate(now) {
    const children = overlayEl?.children || [];
    const height = window.innerHeight;
    const width = overlayEl?.clientWidth || window.innerWidth;
    for (let i = 0; i < children.length; i++) {
      const el = children[i];
      const id = Number(el.dataset.petalId);
      if (Number.isNaN(id)) continue;

      // We don't keep an in-memory list; compute from dataset and stored start timestamp on element via data-born
      if (!el._meta) {
        // reconstruct meta from styles (best effort)
        el._meta = { born: performance.now(), duration: rand(7000, 12000) };
      }
      const meta = el._meta;

      const elapsed = now - meta.born;
      const progress = Math.min(elapsed / meta.duration, 1);

      // To estimate consistent motion, use left and compute x offset
      const leftPct = parseFloat(el.style.left || '0');
      const baseX = (leftPct / 100) * width;

      const sway = parseFloat(el.getAttribute('data-sway') || '0') || rand(40, 120);
      const phase = parseFloat(el.getAttribute('data-phase') || '0') || 0;
      const spin = parseFloat(el.getAttribute('data-spin') || '0') || 0;

      // Vertical position (start slightly above viewport)
      const y = -0.1 * height + progress * (height + 0.25 * height);

      // Horizontal oscillation
      const x = baseX + Math.sin(progress * Math.PI * 2 + phase) * sway;

      // rotation
      const rot = (spin * progress) + (parseFloat(el.getAttribute('data-rotation') || '0'));

      // fade in/out easing
      let opacity = 1;
      if (progress < 0.08) opacity = progress / 0.08;
      if (progress > 0.9) opacity = Math.max(0, (1 - progress) / 0.1);

      el.style.transform = `translate3d(${x - baseX}px, ${y}px, 0) rotate(${rot}deg)`;
      el.style.opacity = String(opacity);

      if (progress >= 1) {
        try { el.remove(); } catch (e) {}
      }
    }
    rafId = requestAnimationFrame(animate);
  }

  let spawnTimer;
  onMount(() => {
    overlayEl = document.querySelector('.sakura-overlay');
    if (!overlayEl) return;

    // respect reduced motion
    const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
    if (mq.matches) return;

    // initialize a couple of petals
    for (let i = 0; i < 6; i++) {
      setTimeout(spawnPetal, i * 120);
    }

    spawnTimer = setInterval(spawnPetal, SPAWN_INTERVAL);
    rafId = requestAnimationFrame(function t(now) { return animate(now); });

    // expose some randomized attributes for animate() to use
    // after DOM node created, update attributes on each petal
    const observer = new MutationObserver(muts => {
      muts.forEach(m => {
        m.addedNodes.forEach(n => {
          if (n instanceof HTMLElement && n.classList.contains('petal')) {
            // set random meta
            n._meta = { born: performance.now(), duration: rand(7000, 14000) };
            n.setAttribute('data-sway', String(rand(40, 160)));
            n.setAttribute('data-phase', String(rand(0, Math.PI * 2)));
            n.setAttribute('data-spin', String(rand(-420, 420)));
            n.setAttribute('data-rotation', String(rand(-140, 140)));
          }
        });
      });
    });
    observer.observe(overlayEl, { childList: true });

    // clean up on destroy
    onDestroy(() => {
      running = false;
      clearInterval(spawnTimer);
      if (rafId) cancelAnimationFrame(rafId);
      observer.disconnect();
      // remove any leftover petals
      overlayEl.querySelectorAll('.petal').forEach(p => p.remove());
    });
  });
</script>

<style>
  :global(.sakura-overlay) {
    pointer-events: none;
    position: fixed;
    inset: 0;
    z-index: 300; /* above content but below modals and some widgets */
    overflow: hidden;
  }

  :global(.petal) {
    position: absolute;
    top: 0;
    left: 0;
    transform-origin: center;
    user-select: none;
    -webkit-user-select: none;
    will-change: transform, opacity;
    transition: opacity 200ms linear;
  }

  @media (prefers-reduced-motion: reduce) {
    :global(.petal) { display: none; }
  }

  @media (max-width: 640px) {
    /* slightly reduce z-index on small screens */
    :global(.sakura-overlay) { z-index: 200; }
  }
</style>

<div class="sakura-overlay" aria-hidden="true"></div>
