---
layout: about
title: about
permalink: /
subtitle: Postdoctoral Scholar, <a href='https://efi.uchicago.edu/'>Enrico Fermi Institute, University of Chicago</a>

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  more_info: >
    <p>Me, in front of the magnet from <a href='https://www2.lbl.gov/Science-Articles/Archive/early-years.html'>EO Lawrence's 27-inch cyclotron</a> </p><p>(photo credit: WH McNeil)</p>

selected_papers: false # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: true # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  # limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

As a postdoctoral scholar at the University of Chicago, I am working with Professor [Young-Kee Kim](https://hep.uchicago.edu/~ykkim/index.shtml) to investigate the intersection between accelerator physics and machine learning.
Particle accelerators are ideal playgrounds for exploring the potential of scientific machine learning.
These systems generate large, complex, and physically-constrained data sets, which can be challenging to analyze using traditional methods.
Machine learning techniques, however, have shown promise in uncovering complex nonlinear patterns and correlations in data which may be used to improve the design and operation of particle accelerators.
In the work I am leading at the University of Chicago, we are integrating machine learning techniques with the physical laws that govern charged particle beams to improve the modeling, instrumentation, control, and optimization of particle accelerators.

As an example of the power of machine learning, the following figure represents the output of a neural network which has been trained to predict the density of a charged particle beam in phase space as it traverses a toy particle accelerator system with the inclusion of space charge forces.
Space charge forces are usually expensive to model in physics-based simulation tools.
However, after the neural network has been trained on sparse calculated results, it is able to query the properties of the beam as system parameters are varied in an efficient and differentiable manner.

{:refdef: style="text-align: center;"}
![A visualization of isosurfaces of a beam distribution function solved for using scientific machine learning](/assets/img/pinn-beam-isosurfaces.png){: width="500" }
{: refdef}

In addition to my work on scientific machine learning, my interests include photocathode physics for particle accelerator applications.
Photocathodes are used to generate high-brightness electron beams which are necessary for some applications of particle accelerators, such as free electron lasers and ultrafast electron diffraction.
The brightness of electron beams produced by the photocathode is highly dependent on the choice of photocathode material and in particular a material-dependent property called the mean transverse energy.
Measuring the mean tranverse energy of photocathodes and devloping models that can predict this metric based on other material properties and understand how real-world problems can affect models are important goals in accelerator physics.

My work in the area of photocathode physics is related to exploiting the electronic band structure of semiconductors to limit the mean transverse energy of the photocathode and explore nonlinear photoemission processes via nanopatterned surfaces as a mechanism to shrink the photocathode's source size (see image below).
My past work has also shown how nonlinear processes can be enhanced using nanoscale patterning of metal surfaces to improve the efficiency of generating short pulses of electrons.
Additionally, I created and maintain an open repository of published data related to photocathode materials.
[The Photocathode Database](https://photocathodes.io) exists to connect researchers with published measurements on a variety of photocathode materials.
Although it is currently small, I hope it will continue to grow in usefulness as I continue to digitize and add data to it.
I am always looking for help to maintain this resources and interested parties should contact me for information on how to get started.


{:refdef: style="text-align: center;"}
![A scanning electron micrograph of a nanopatterned spiral photocathode proposed for source size reduction](/assets/img/spiral-plasmonic-lens.png){: width="500" }
{: refdef}