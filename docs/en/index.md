---
layout: home
title: CKCsec Wiki
titleTemplate: A security research knowledge base

hero:
  name: "CKCsec Wiki"
  text: "A bilingual knowledge base for security practitioners"
  tagline: "Curated notes and practical references for Web security, blockchain security, CTF, red team research, and defensive operations."
  actions:
    - theme: brand
      text: Start Reading
      link: /en/about/
    - theme: alt
      text: 中文
      link: /
    - theme: alt
      text: GitHub
      link: https://github.com/ckcsec/wiki-vitepress
  image:
    src: /images/ckcsec-docs-terminal.svg
    alt: CKCsec Wiki bilingual documentation workspace

features:
  - title: Open Knowledge
    details: The project keeps its site source and knowledge base public so the community can learn, review, and improve it together.
  - title: Practical Security
    details: Content focuses on hands-on notes across Web security, blockchain security, CTF, red team topics, and incident response.
  - title: Bilingual Growth
    details: English entry pages are now available, with deeper article translations planned as the knowledge base evolves.
---

<style>
:root {
  --vp-home-hero-name-color: transparent;
  --vp-home-hero-name-background: -webkit-linear-gradient(120deg, #0ea5e9 20%, #14b8a6);
  --vp-home-hero-image-background-image: linear-gradient(135deg, rgba(14, 165, 233, 0.28), rgba(20, 184, 166, 0.28));
  --vp-home-hero-image-filter: blur(42px);
}

@media (min-width: 640px) {
  :root {
    --vp-home-hero-image-filter: blur(52px);
  }
}

@media (min-width: 960px) {
  :root {
    --vp-home-hero-image-filter: blur(64px);
  }
}
</style>
