---
layout: page
permalink: /repositories/
title: Repositories
description:
nav: true
nav_order: 4
---

I am committed to the principles of open science and the use of open-source scientific software.
My own released projects are available to download on my github account under the username [@electronsandstuff](https://github.com/electronsandstuff).
Additionally, I operate [The Photocathode Database](https://photocathodes.io), a repository for published measurements on photocathode materials.
I encourage others to contribute to my projects, whether by submitting bug reports, helping to digitize scientific data, making feature requests, or contributing code.

The following repositories are a couple of the projects that I have released.
Follow the links and take a look around.

{% if site.data.repositories.github_repos %}

## GitHub Repositories

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% for repo in site.data.repositories.github_repos %}
    {% include repository/repo.liquid repository=repo %}
  {% endfor %}
</div>
{% endif %}
