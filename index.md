---
layout: default
title: Home
hide_header: true
---

<div class="hero">
  <div class="eyebrow">Security research and Stuff</div>
  <p class="lede">How intrusions actually happen, how defenses actually respond, and where the gap between the two quietly opens.</p>
  {% assign post_count = site.posts | size %}
  <div class="logline">{{ post_count }} write-ups <span>·</span> research on forensics, vulnerabilities, and threat groups</div>
</div>

<div class="filters-label">Filter by research area</div>
<div class="filters">
  <button class="filter-tag active" onclick="filterByTag('all')">all</button>
  <button class="filter-tag" onclick="filterByTag('forensics')">forensics</button>
  <button class="filter-tag" onclick="filterByTag('vulnerability')">vulnerability</button>
  <button class="filter-tag" onclick="filterByTag('threat_groups')">threat_groups</button>
  <button class="filter-tag" onclick="filterByTag('research')">research</button>
  <button class="filter-tag" onclick="filterByTag('mongodb')">mongodb</button>
  <button class="filter-tag" onclick="filterByTag('oracle_ebs')">oracle_ebs</button>
  <button class="filter-tag" onclick="filterByTag('sap_netweaver')">sap_netweaver</button>
  <button class="filter-tag" onclick="filterByTag('fortinet')">fortinet</button>
  <button class="filter-tag" onclick="filterByTag('panos')">panos</button>
  <button class="filter-tag" onclick="filterByTag('sonicwall')">sonicwall</button>
  <button class="filter-tag" onclick="filterByTag('ivanti')">ivanti</button>
  <button class="filter-tag" onclick="filterByTag('cisco')">cisco</button>
  <button class="filter-tag" onclick="filterByTag('citrix')">citrix</button>
  <button class="filter-tag" onclick="filterByTag('wsus')">wsus</button>
</div>

<div class="ledger" id="research-posts">
{% for post in site.posts %}
  {% assign tags_lower = "" %}
  {% for t in post.tags %}{% assign tags_lower = tags_lower | append: t | append: "," %}{% endfor %}
  {% assign is_critical = false %}
  {% if post.cvss_score and post.cvss_score >= 9 %}{% assign is_critical = true %}{% endif %}
  <a class="research-post{% if is_critical %} critical{% endif %}" href="{{ site.baseurl }}{{ post.url }}" data-tags="{{ tags_lower | downcase }}">
    <div class="meta">
      <span>{{ post.date | date: "%Y-%m-%d" }}</span>
      {% if post.stamp_label %}
      <span class="stamp {% if is_critical %}red{% else %}amber{% endif %}">{{ post.stamp_label }}</span>
      {% endif %}
    </div>
    <div class="row">
      <div class="col-text">
        <h4>{{ post.title }}</h4>
        {% if post.subtitle %}<p class="post-dek">{{ post.subtitle }}</p>{% endif %}
        <div class="post-tags">
          {% for tag in post.tags %}<span class="tag">{{ tag }}</span>{% endfor %}
        </div>
      </div>
      <div class="post-image" role="img" aria-label="Thumbnail for {{ post.title }}" style="background-image:url('{{ site.baseurl }}{{ post.header_image }}');"></div>
    </div>
  </a>
{% endfor %}
</div>

### Research Disclaimer
<span class="disclaimer">
Research published is provided for educational purposes. Findings reflect observed behavior in specific environments and should not be interpreted as universal truth, vendor endorsement, or operational guidance. Techniques discussed may be incomplete, ineffective, or rendered obsolete without notice. Readers are expected to apply judgment, skepticism, and basic security hygiene.
</span>

<script>
function filterByTag(tag) {
  const posts = document.querySelectorAll('.research-post');
  const filterTags = document.querySelectorAll('.filter-tag');

  filterTags.forEach(ft => {
    ft.classList.toggle('active', ft.textContent.trim() === tag);
  });

  posts.forEach(post => {
    if (tag === 'all') {
      post.style.display = 'block';
      return;
    }
    const tags = post.getAttribute('data-tags').toLowerCase();
    post.style.display = tags.includes(tag.toLowerCase()) ? 'block' : 'none';
  });
}
</script>
