---
layout: about
title: About
permalink: /
subtitle: Accelerator Physicist, Stealth Startup

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  address: >
    <p>The magnet from <a href='https://www2.lbl.gov/Science-Articles/Archive/early-years.html'>EO Lawrence's 27-inch cyclotron</a> </p><p>(photo credit: WH McNeil)</p>
news: true  # includes a list of news items
selected_papers: false # includes a list of papers marked as "selected={true}"
social: true  # includes social icons at the bottom of the page
---

Particle accelerators are an ideal playground to explore the potential of scientific machine learning.
These systems generate large, complex, and physically-constrained data sets, which can be challenging to analyze using traditional methods.
Machine learning techniques, however, have shown promise in uncovering complex nonlinear patterns and correlations in data and we may be able to use them to improve the way particle accelerators are designed and operated.
In my work, I am integrating machine learning techniques with the physical laws that govern charged particle beams to improve the modeling, instrumentation, control, and optimization of particle accelerators.

As an example of the power of machine learning, the following figure represents the output of a neural network which has been trained to predict the shape of a particle beam in phase space as it traverses a toy particle accelerator.
Nonlinear space charge forces degrade the beam's "emittance" (in plain language, quality) over time.
Space charge forces are usually expensive to model in physics-based simulation tools.
However, after the neural network has been trained on sparse calculated results, it is able to query the properties of the beam as system parameters are varied in an efficient and differentiable manner.

{:refdef: style="text-align: center;"}
![A visualization of a particle beam's distribution function solved for using scientific machine learning](/assets/img/uniform-focusing-channel-beam.gif){: width="256" }
{: refdef}

In addition to my work on scientific machine learning, my interests include photocathode physics for particle accelerator applications.
Photocathodes are used to generate high-brightness electron beams which are necessary for some applications of particle accelerators, such as [free electron lasers](https://lcls.slac.stanford.edu/lcls-ii) and [ultrafast electron diffraction](https://en.wikipedia.org/wiki/Ultrafast_electron_diffraction).
The brightness of electron beams produced by the photocathode is highly dependent on the choice of photocathode material and in particular a material-dependent property called the mean transverse energy.
Measuring the mean tranverse energy of photocathodes and devloping models that can predict this metric based on other material properties and understand how real-world problems can affect models are important goals in accelerator physics.

My work in the area of photocathode physics is related to exploiting the electronic band structure of semiconductors to limit the mean transverse energy of the photocathode and explore nonlinear photoemission processes via nanopatterned surfaces as a mechanism to shrink the photocathode's source size (see image below).
My past work has also shown how nonlinear processes can be enhanced using nanoscale patterning of metal surfaces to improve the efficiency of generating short pulses of electrons.
Additionally, I created and maintain an open repository of published data related to photocathode materials.
[The Photocathode Database](https://photocathodes.io) exists to connect researchers with published measurements on a variety of photocathode materials.
Although it is currently small, I hope it will continue to grow in usefulness as I continue to digitize and add data to it.
I am always looking for help to maintain this resources and interested parties should contact me for information on how to get started.


{:refdef: style="text-align: center;"}
![A scanning electron micrograph of a nanopatterned spiral photocathode proposed for source size reduction](/assets/img/spiral-plasmonic-lens.png){: width="400" }
{: refdef}