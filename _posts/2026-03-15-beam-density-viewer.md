---
layout: post
title: Particle Beam Implicit Models
date: 2026-03-15 11:12:00-0400
description: An interactive viewer for exploring beam phase-space density predictions from a neural network surrogate model.
tags:
categories:
related_posts: false
---

Placeholder text

<div id="density-viewer">

<div id="status" style="padding:10px;border-radius:4px;margin-bottom:20px;background:#fff3cd;color:#856404;">Loading model...</div>

<div class="controls" style="background:white;padding:20px;border-radius:8px;box-shadow:0 2px 4px rgba(0,0,0,0.1);margin-bottom:20px;">
  <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;">
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">
        s (position): <span style="font-family:monospace;color:#007bff;" id="dv-s_val">0.0</span>
        <button id="dv-play" style="margin-left:10px;padding:2px 10px;border:1px solid #007bff;border-radius:4px;background:white;color:#007bff;cursor:pointer;font-size:0.85em;">&#9654; Play</button>
      </label>
      <input type="range" id="dv-s" min="0" max="300" step="1" value="0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">twiss_beta: <span style="font-family:monospace;color:#007bff;" id="dv-twiss_beta_val">3.0</span></label>
      <input type="range" id="dv-twiss_beta" min="3" max="8" step="0.1" value="3.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">twiss_alpha: <span style="font-family:monospace;color:#007bff;" id="dv-twiss_alpha_val">0.0</span></label>
      <input type="range" id="dv-twiss_alpha" min="-0.5" max="0.5" step="0.1" value="0.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">supergaussian_n: <span style="font-family:monospace;color:#007bff;" id="dv-supergaussian_n_val">2.0</span></label>
      <input type="range" id="dv-supergaussian_n" min="1" max="10" step="0.5" value="2.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">perveance: <span style="font-family:monospace;color:#007bff;" id="dv-perveance_val">0.00001</span></label>
      <input type="range" id="dv-perveance" min="0.00001" max="0.0001" step="0.00001" value="0.0001" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">lattice_k: <span style="font-family:monospace;color:#007bff;" id="dv-lattice_k_val">0.125</span></label>
      <input type="range" id="dv-lattice_k" min="0.125" max="0.333" step="0.002" value="0.125" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">emittance: <span style="font-family:monospace;color:#007bff;" id="dv-emittance_val">1.0</span> μm</label>
      <input type="range" id="dv-emittance" min="0.1" max="1.0" step="0.05" value="1.0" style="width:100%;">
    </div>
  </div>
</div>

<div id="dv-plot" style="background:white;border-radius:8px;box-shadow:0 2px 4px rgba(0,0,0,0.1);"></div>

</div>

<script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/onnxruntime-web@1.17.0/dist/ort.min.js"></script>
<script>
(function() {
  let session = null;
  let isComputing = false;
  let isPlaying = false;
  let playStartTime = null;
  const PLAY_DURATION_MS = 7500;
  const FRAME_MS = 1000 / 24;

  const MODEL_URL = '/assets/html/2026-03-15-beam-density-viewer/model.onnx';
  const RESOLUTION = 96;
  const X_RANGE = 1.5e-2;
  const XP_RANGE = 3e-3;

  const paramIds = ['s', 'twiss_beta', 'twiss_alpha', 'supergaussian_n', 'perveance', 'lattice_k', 'emittance'];

  Plotly.newPlot('dv-plot', [{
    z: [[0]],
    type: 'heatmap',
    colorscale: 'Viridis',
    colorbar: { title: 'Density (k/mm/μm)' }
  }], {
    title: 'Beam Density Distribution',
    xaxis: { title: 'x (mm)' },
    yaxis: { title: "x' (mrad)" },
    width: 800,
    height: 600,
  });

  paramIds.forEach(param => {
    const slider = document.getElementById('dv-' + param);
    const valEl = document.getElementById('dv-' + param + '_val');
    slider.addEventListener('input', () => {
      let v = parseFloat(slider.value);
      if (param === 'emittance') valEl.textContent = v.toFixed(2);
      else if (param === 'perveance') valEl.textContent = v.toFixed(5);
      else valEl.textContent = v.toPrecision(3);
      scheduleUpdate();
    });
  });

  let updateTimeout = null;
  function scheduleUpdate() {
    if (updateTimeout) clearTimeout(updateTimeout);
    updateTimeout = setTimeout(updatePlot, 10);
  }

  const playBtn = document.getElementById('dv-play');
  playBtn.addEventListener('click', () => {
    isPlaying = !isPlaying;
    playBtn.textContent = isPlaying ? '⏸ Pause' : '▶ Play';
    if (isPlaying) {
      playStartTime = performance.now();
      playFrame();
    }
  });

  function playFrame() {
    if (!isPlaying) return;
    const slider = document.getElementById('dv-s');
    const valEl = document.getElementById('dv-s_val');
    const elapsed = (performance.now() - playStartTime) % PLAY_DURATION_MS;
    const s = Math.round((elapsed / PLAY_DURATION_MS) * parseInt(slider.max));
    slider.value = s;
    valEl.textContent = s.toPrecision(3);
    const frameStart = performance.now();
    updatePlot().then(() => {
      if (!isPlaying) return;
      const frameElapsed = performance.now() - frameStart;
      setTimeout(playFrame, Math.max(0, FRAME_MS - frameElapsed));
    });
  }

  async function loadModel() {
    const status = document.getElementById('status');
    try {
      session = await ort.InferenceSession.create(MODEL_URL);
      status.style.background = '#d4edda';
      status.style.color = '#155724';
      status.textContent = 'Model ready';
      updatePlot();
    } catch (err) {
      status.style.background = '#f8d7da';
      status.style.color = '#721c24';
      status.textContent = 'Error loading model: ' + err.message;
      console.error(err);
    }
  }

  async function updatePlot() {
    if (!session) return;
    if (isComputing) return;
    isComputing = true;
    try {
      const s = parseFloat(document.getElementById('dv-s').value);
      const twiss_beta = parseFloat(document.getElementById('dv-twiss_beta').value);
      const twiss_alpha = parseFloat(document.getElementById('dv-twiss_alpha').value);
      const supergaussian_n = parseFloat(document.getElementById('dv-supergaussian_n').value);
      const perveance = parseFloat(document.getElementById('dv-perveance').value);
      const lattice_k = parseFloat(document.getElementById('dv-lattice_k').value);
      const emittance = parseFloat(document.getElementById('dv-emittance').value) * 1e-6;
      const xpValues = [];
      for (let j = 0; j < RESOLUTION; j++) {
        xpValues.push(-XP_RANGE + (2 * XP_RANGE * j) / (RESOLUTION - 1));
      }

      const halfRes = RESOLUTION / 2;
      const xValuesHalf = [];
      for (let i = 0; i < halfRes; i++) {
        xValuesHalf.push((X_RANGE * i) / (halfRes - 1));
      }
      const batchSize = halfRes * RESOLUTION;
      const inputData = new Float32Array(batchSize * 9);
      let idx = 0;
      for (let j = 0; j < RESOLUTION; j++) {
        for (let i = 0; i < halfRes; i++) {
          inputData[idx++] = xValuesHalf[i];
          inputData[idx++] = xpValues[j];
          inputData[idx++] = s;
          inputData[idx++] = twiss_beta;
          inputData[idx++] = twiss_alpha;
          inputData[idx++] = supergaussian_n;
          inputData[idx++] = perveance;
          inputData[idx++] = lattice_k;
          inputData[idx++] = emittance;
        }
      }
      const inputTensor = new ort.Tensor('float32', inputData, [batchSize, 9]);
      const results = await session.run({ input: inputTensor });
      const outputData = results.density.data;

      const zData = [];
      for (let j = 0; j < RESOLUTION; j++) {
        const row = [];
        const mirrorJ = RESOLUTION - 1 - j;
        for (let i = halfRes - 1; i >= 0; i--) row.push(outputData[mirrorJ * halfRes + i] * 1e-9);
        for (let i = 0; i < halfRes; i++) row.push(outputData[j * halfRes + i] * 1e-9);
        zData.push(row);
      }
      const xDisplay = [];
      for (let i = halfRes - 1; i >= 0; i--) xDisplay.push(-xValuesHalf[i] * 1e3);
      for (let i = 0; i < halfRes; i++) xDisplay.push(xValuesHalf[i] * 1e3);

      const xpDisplay = xpValues.map(v => v * 1e3);
      Plotly.update('dv-plot', { z: [zData], x: [xDisplay], y: [xpDisplay] }, {
        'xaxis.range': [-X_RANGE * 1e3, X_RANGE * 1e3],
        'yaxis.range': [-XP_RANGE * 1e3, XP_RANGE * 1e3]
      });
    } catch (err) {
      console.error('Inference error:', err);
    }
    isComputing = false;
  }

  loadModel();
})();
</script>
