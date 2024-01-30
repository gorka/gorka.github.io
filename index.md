---
layout: default
title: "Hello!"
---

<section id="notes" class="">
  <h2>📝 Notes</h2>

  <ul class="list-inside list-disc">
  {% for note in site.notes %}
    <li><a href="{{ note.url }}">{{ note.title }}</a></li>
  {% endfor %}
  </ul>
</section>
