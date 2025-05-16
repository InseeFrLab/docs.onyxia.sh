---
icon: memo
---

# Custom Pages

Your can host your custom documentation pages directly within your Onyxia instance.  \
It's perfect if you want to provide some custom instruction for getting starter or write your own step by step tutorial specifically tailored to your users. &#x20;

\<VIDEO DEMO>

## How it work

Your document have to be Markdown files. They will be renderd as HTML within Onyxia.\
Your documents should be hosted directly within your Onyxia instance, they can't be external: You must include them in the ZIP file that you provide as CUSTOM\_RESOURCES.  \
More info in theme and branding documentation page.  \
You can point to one of your Mardown document from every customisable link, in the header, leftbar, fooder and other.  \
You can point to markdown document from other markdown documents.  \
\
Example:

Assuming we have the following document inside the custom-resources.zip that we provide to our onyxia instalce:

```
/onboarding_en.md
/onboarding_fr.md
```

We can write: &#x20;

<pre class="language-yaml" data-title="onyxia/values.yaml"><code class="lang-yaml">onyxia:
  web:
    env:
<strong>      CUSTOM_RESOURCES: "https://.../custom-resources.zip"
</strong>      HEADER_TEXT_BOLD: My Organization
      HEADER_TEXT_FOCUS: Datalab
      HEADER_LINKS: |
        [
          {
            label: {
              en: "Onboarding Guide",
              fr: "Guide d'intégration",
            },
            icon: "School",
            url: {
<strong>              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>            },
          }
        ]
      FOOTER_LINKS: |
        [
          {
            label: {
              en: "Onboarding Guide",
              fr: "Guide d'intégration",
            },
            icon: "School",
            url: {
<strong>              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>            },
          }
        ]
      HOMEPAGE_BELOW_HERO_TEXT: |
        {
<strong>          en: "See our [onboarding guide](%PUBLIC_URL%/custom-resources/onboarding_en.md)",
</strong><strong>          fr: "Consultez notre [guide d'intégration](%PUBLIC_URL%/custom-resources/onboarding_fr.md)"
</strong>        }
      HOMEPAGE_CALL_TO_ACTION_BUTTON: |
        {
          "label": {
<strong>            en: "Read our get started guide",
</strong><strong>            fr: "Lire notre guide de démarrage",
</strong>          },
          "startIcon": "School",
          "url": {
<strong>              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>          }
        }
      TERMS_OF_SERVICES: "%PUBLIC_URL%/custom-resources/tos_fr.md"
</code></pre>
