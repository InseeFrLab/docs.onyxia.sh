---
icon: memo
---

# Custom Pages

You can host your own custom documentation pages directly within your Onyxia instance.\
This is ideal if you want to provide onboarding instructions or write step-by-step tutorials specifically tailored to your users.

\<VIDEO DEMO>

## How It Works

Your documentation must consist of Markdown files. These files will be rendered as HTML within the Onyxia UI.\
The documents must be hosted within your Onyxia instance; external links are not supported. You need to include them in the `custom-resources.zip` file, provided through the `CUSTOM_RESOURCES` configuration key.\
More details are available in the [theme and branding documentation](../theme-and-branding.md).

You can link to your Markdown files from any customizable section of the interface: header, sidebar, footer, and even from other Markdown files.

### Example

Assume we include the following files in `custom-resources.zip`:

```
/onboarding_en.md
/onboarding_fr.md
```

We can reference them in our configuration:

<pre class="language-yaml" data-title="onyxia/values.yaml"><code class="lang-yaml">onyxia:
  web:
    env:
      CUSTOM_RESOURCES: "https://.../custom-resources.zip"
      HEADER_TEXT_BOLD: My Organization
      HEADER_TEXT_FOCUS: Datalab
      HEADER_LINKS: |
        [
          {
            label: {
              en: "Onboarding Guide",
              fr: "Guide d'intégration"
            },
            icon: "School",
            url: {
              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
            }
          }
        ]
      FOOTER_LINKS: |
        [
          {
            label: {
              en: "Onboarding Guide",
              fr: "Guide d'intégration"
            },
            icon: "School",
            url: {
              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
            }
          }
        ]
      HOMEPAGE_BELOW_HERO_TEXT: |
        {
          en: "See our [onboarding guide](%PUBLIC_URL%/custom-resources/onboarding_en.md)",
          fr: "Consultez notre [guide d'intégration](%PUBLIC_URL%/custom-resources/onboarding_fr.md)"
        }
      HOMEPAGE_CALL_TO_ACTION_BUTTON: |
        {
          label: {
            en: "Read our get started guide",
            fr: "Lire notre guide de démarrage"
          },
          startIcon: "School",
          url: {
            en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
            fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
          }
        }
      TERMS_OF_SERVICES: "%PUBLIC_URL%/custom-resources/tos_fr.md"
</code></pre>
