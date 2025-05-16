---
icon: memo
---

# Custom Pages

You can host your own custom documentation pages directly within your Onyxia instance.\
This is ideal if you want to provide onboarding instructions or write step-by-step tutorials specifically tailored to your users.

{% embed url="https://youtu.be/aQVu-vsf51w" %}

## How It Works

Your documentation must consist of Markdown files. These files will be rendered as HTML within the Onyxia UI.\
The documents must be hosted within your Onyxia instance; external links are not supported. You need to include them in the `custom-resources.zip` file, provided through the `CUSTOM_RESOURCES` configuration key.\
More details are available in the [theme and branding documentation](theme.md).

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
<strong>      CUSTOM_RESOURCES: "https://.../custom-resources.zip"
</strong>      HEADER_TEXT_BOLD: My Organization
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
<strong>              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>            }
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
<strong>              en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>              fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>            }
          }
        ]
      HOMEPAGE_BELOW_HERO_TEXT: |
        {
<strong>          en: "See our [onboarding guide](%PUBLIC_URL%/custom-resources/onboarding_en.md)",
</strong><strong>          fr: "Consultez notre [guide d'intégration](%PUBLIC_URL%/custom-resources/onboarding_fr.md)"
</strong>        }
      HOMEPAGE_CALL_TO_ACTION_BUTTON: |
        {
          label: {
            en: "Read our get started guide",
            fr: "Lire notre guide de démarrage"
          },
          startIcon: "School",
          url: {
<strong>            en: "%PUBLIC_URL%/custom-resources/onboarding_en.md",
</strong><strong>            fr: "%PUBLIC_URL%/custom-resources/onboarding_fr.md"
</strong>          }
        }
      TERMS_OF_SERVICES: "%PUBLIC_URL%/custom-resources/tos_fr.md"
</code></pre>

Example of Mardown document

{% code title="onboarding_en.md" %}
````markdown
# This is a test document in english

This could be for example a guide specific to your Onyxia instance.  

## It's standard markdown

You can embed images, including with HTML syntax:  

<img src="%PUBLIC_URL%/custom-resources/preview.png" width="100%">  

You can render code snippets:  

```bash
echo "Hello world"
```

You can also link to pages of your instance: [Catalog](/catalog).

You can link to [another document](%PUBLIC_URL%/custom-resources/onboarding_sub_en.md).

<a href="%PUBLIC_URL%/launcher/ide/rstudio?name=rstudio&version=2.3.2&s3=region-ec97c721&resources.limits.cpu=«22700m»&autoLaunch=true">
    <img height=20 src="https://user-images.githubusercontent.com/6702424/173724486-30b6232a-c5d2-40da-a0cc-4d4a11824135.png">
</a>
````
{% endcode %}
