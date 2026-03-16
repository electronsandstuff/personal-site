---
layout: post
title: Learning Implicit Particle Beam Representations
date: 2026-03-15 11:12:00-0400
description: Making a neural network pretend to be a particle beam.
tags:
categories:
related_posts: false
---


In accelerator physics, a key representation of particle beams is their density in phase space.
In our 3D world, this takes the form of the function $\rho(x, p_x, y, p_y, z, p_z)$, but for the purposes of a machine learning experiment we may consider data from a simple one dimensional system and the corresponding beam $\rho(x, p_x)$.
Here, data on the beam takes the form of images encoding density within some bounding box around the beam.
These images could be learned directly, but it is interesting to instead learn the function $\rho(x, p_x, \vec{\Theta})$ directly for some system parameters $\vec{\Theta}$.
This is called an implicit neural representation and has [some writing](https://arxiv.org/pdf/2411.03688) on it already.

Using a particle tracking code, I have already generated a dataset of 1D electron beams in a uniform focusing channel with space charge for a variety of system and initial parameters.
The initial beam was [supergaussian]({% post_url /_posts/2025-09-16-sampling-from-supergaussian %}) and I varied the following (with some descriptions for those not used to accelerator physics jargon).
- Initial Twiss beta (the initial ratio of the beam's size in $x$ and $p_x$)
- Initial Twiss alpha (the initial correlation between $x$ and $p_x$)
- Initial emittance (beam's initial area in phase space)
- Supergaussian "n" of initial distribution (how "sharp" is the initial distribution)
- Beam perveance (bunch charge, controls amount of space charge forces... ie the repulsion of the beam away from itself.)
- Channel focusing strength (how strongly does the external focusing element affect the beam)
Here are some example images.

<p align="center">
<img src="/assets/img/tech_notes/2026-03-15-beam/data.png" alt="Particle beam phase spaces" width="90%" style="max-width: 600px">
</p>

Using an MLP with $\exp(x)$ tacked onto the end, I was able to successfully learn the implicit representation of the data.
This simple model makes sense as all of the densities should look a bit like supergaussians $A\cdot\exp\left[-\left(x^2 + p_x^2\right)^n\right]$ with various simple nonlinear mappings of $x$ and $p_x$ applied.
The MLP should be ok at learning the stuff inside $\exp(\cdots)$.
This ended up taking more compute than I was expecting for the system, but I suppose the number of parameters ended up being ambitious and the number of training samples is large considering each pixel in the images counts.
Training went well with the learning rate getting dropped after successive rounds.
I found that a fairly large learning rate was needed to get the model to start to look "beam-like" at all and then a lower learning rate is needed to begin representing details in the image.

<p align="center">
<img src="/assets/img/tech_notes/2026-03-15-beam/training.jpg" alt="Training curves" width="75%" style="max-width: 400px">
</p>

As an experiment in deploying models, I am embedding a viewer directly into this post.
Try hitting "play" in the window below.
The plots being shown are not video, these are the output of a neural network with inference being done directly in your browser.
Go ahead and change the knobs and explore the dataset.

This simple model has some "stripe" artifacts in it which I would like to work out.
I believe regularization will help.

<div id="density-viewer">

<div id="status" style="padding:10px;border-radius:4px;margin-bottom:20px;display:none;"></div>

<div class="controls" style="background:white;padding:20px;border-radius:8px;box-shadow:0 2px 4px rgba(0,0,0,0.1);margin-bottom:20px;">
  <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;">
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">
        Position (s): <span style="font-family:monospace;color:#007bff;" id="dv-s_val">0.0</span><span style="font-family:monospace;color:#007bff;"> m</span>
        <button id="dv-play" style="margin-left:10px;padding:2px 10px;border:1px solid #007bff;border-radius:4px;background:white;color:#007bff;cursor:pointer;font-size:0.85em;">&#9654; Play</button>
      </label>
      <input type="range" id="dv-s" min="0" max="300" step="1" value="0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Initial Twiss Beta: <span style="font-family:monospace;color:#007bff;" id="dv-twiss_beta_val">3.0</span><span style="font-family:monospace;color:#007bff;"> m</span></label>
      <input type="range" id="dv-twiss_beta" min="3" max="8" step="0.1" value="3.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Initial Twiss Alpha: <span style="font-family:monospace;color:#007bff;" id="dv-twiss_alpha_val">0.0</span></label>
      <input type="range" id="dv-twiss_alpha" min="-0.5" max="0.5" step="0.1" value="0.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Initial Supergaussian `n`: <span style="font-family:monospace;color:#007bff;" id="dv-supergaussian_n_val">2.0</span></label>
      <input type="range" id="dv-supergaussian_n" min="1" max="10" step="0.5" value="2.0" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Perveance (k): <span style="font-family:monospace;color:#007bff;" id="dv-perveance_val">1.0e-4</span></label>
      <input type="range" id="dv-perveance" min="0.00001" max="0.0001" step="0.00001" value="0.0001" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Lattice Focusing k: <span style="font-family:monospace;color:#007bff;" id="dv-lattice_k_val">0.125</span><span style="font-family:monospace;color:#007bff;"> m⁻¹</span></label>
      <input type="range" id="dv-lattice_k" min="0.125" max="0.333" step="0.002" value="0.125" style="width:100%;">
    </div>
    <div>
      <label style="display:block;margin-bottom:5px;font-weight:500;color:#555;">Initial Emittance: <span style="font-family:monospace;color:#007bff;" id="dv-emittance_val">1.0</span><span style="font-family:monospace;color:#007bff;"> μm</span></label>
      <input type="range" id="dv-emittance" min="0.1" max="1.0" step="0.05" value="1.0" style="width:100%;">
    </div>
  </div>
</div>

<div id="dv-plot" style="background:white;border-radius:8px;box-shadow:0 2px 4px rgba(0,0,0,0.1);width:100%;"></div>

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
    colorbar: { title: { text: 'Density<br>(k/mm/μm)', side: 'top', font: { size: 10 } }, thickness: 15 }
  }], {
    title: 'Beam Density Distribution',
    xaxis: { title: 'x (mm)' },
    yaxis: { title: "x' (mrad)" },
    height: 500,
  }, { responsive: true });

  paramIds.forEach(param => {
    const slider = document.getElementById('dv-' + param);
    const valEl = document.getElementById('dv-' + param + '_val');
    slider.addEventListener('input', () => {
      let v = parseFloat(slider.value);
      if (param === 'emittance') valEl.textContent = v.toFixed(2);
      else if (param === 'perveance') valEl.textContent = v.toExponential(1);
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
      updatePlot();
    } catch (err) {
      status.style.display = 'block';
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
