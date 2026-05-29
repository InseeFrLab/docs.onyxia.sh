---
description: >-
  Customize your Onyxia instance with your assets and your colors, make it your
  own!
icon: palette
---

# Theme and branding

{% embed url="https://youtu.be/NrVuVXsbloA" %}

The full documentation of the available parameter can be found here:

{% embed url="https://github.com/InseeFrLab/onyxia/blob/main/web/.env" %}

## Theme Galery

Here is a galery of theme that you can try out.

{% hint style="info" %}
If you want to test theses theme in your local dev env (as shown in the video) download the ZIP file specified as `CUSTOM_RESOURCES` and extract it in **web/public/custom-resources**.
{% endhint %}

{% tabs %}
{% tab title="Ultraviolet" %}
<figure><img src="../.gitbook/assets/image (57).png" alt=""><figcaption><p>Light Mode</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption><p>Dark mode</p></figcaption></figure>

{% code title="values.yaml" %}
```yaml
onyxia:
  web:
    env:
      #ONYXIA_API_URL: https://datalab.sspcloud.fr/api
      CUSTOM_RESOURCES: "https://www.sspcloud.fr/ultraviolet/custom-resources.zip"
      FONT: |
        { 
          fontFamily: "Geist", 
          dirUrl: "%PUBLIC_URL%/custom-resources/fonts/Geist", 
          "400": "Geist-Regular.woff2",
          "500": "Geist-Medium.woff2",
          "600": "Geist-SemiBold.woff2",
          "700": "Geist-Bold.woff2"
        }
      PALETTE_OVERRIDE: |
        {
          focus: {
            main: "#067A76",
            light: "#0AD6CF",
            light2: "#AEE4E3"
          },
          dark: {
            main: "#2D1C3A",
            light: "#4A3957",
            greyVariant1: "#22122E",
            greyVariant2: "#493E51",
            greyVariant3: "#918A98",
            greyVariant4: "#C0B8C6"
          },
          light: {
            main: "#F7F5F4",
            light: "#FDFDFC",
            greyVariant1: "#E6E6E6",
            greyVariant2: "#C9C9C9",
            greyVariant3: "#9E9E9E",
            greyVariant4: "#747474"
          }
        }
      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/custom-resources/preview.png"
```
{% endcode %}
{% endtab %}

{% tab title="SSPCloud" %}
<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption><p>Light Mode</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (59).png" alt=""><figcaption><p>Dark Mode</p></figcaption></figure>

{% code title="onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      #ONYXIA_API_URL: https://datalab.sspcloud.fr/api
      CUSTOM_RESOURCES: "https://www.sspcloud.fr/onyxia-theme-sspcloud-v0.zip"
      GLOBAL_ALERT: |
        {
          severity: "success",
          message: {
            en: "If you like the platform, you can give us a ⭐️ [on GitHub](https://github.com/InseeFrLab/onyxia). Thank you very much! 😊",
            fr: "Si vous aimez la plateforme, vous pouvez nous mettre une ⭐️ [sur GitHub](https://github.com/InseeFrLab/onyxia). Merci beaucoup ! 😊",
          }
        }
      DISABLE_PERSONAL_INFOS_INJECTION_IN_GROUP: true
      TERMS_OF_SERVICES: |
        {
          en: "%PUBLIC_URL%/custom-resources/tos_en.md",
          fr: "%PUBLIC_URL%/custom-resources/tos_fr.md"
        }
      HEADER_LINKS: |
        [
          {
            label: {
              en: "Tutorials",
              fr: "Tutoriels",
              "zh-CN": "教程",
              fi: "Opastus",
              no: "Opplæring",
              it: "Tutorial",
              nl: "Zelfstudie"
            },
            icon: "https://www.sspcloud.fr/trainings.svg",
            url: "https://www.sspcloud.fr/formation"
          },
          {
            label: "AI Chat",
            icon: "SmartToy",
            url: "https://llm.lab.sspcloud.fr"
          },
          {
            "label": {
              "en": "Contact us",
              "fr": "Contactez nous"
            },
            "icon": "Support",
            "url": "https://join.slack.com/t/3innovation/shared_invite/zt-1bo6y53oy-Y~zKzR2SRg37pq5oYgiPuA"
          }
        ]
      HOMEPAGE_CALL_TO_ACTION_BUTTON_AUTHENTICATED: |
        {
          "label": {
            "fr": "Nouvel utilisateur du datalab ?",
            "en": "New user of the datalab?",
            "zh-CN": "数据实验室新用户？",
            "fi": "Uusi datalabin käyttäjä?",
            "no": "Ny bruker av datalaben?",
            "it": "Nuovo utente del datalab?",
            "nl": "Nieuwe gebruiker van het datalab?"
          },
          "startIcon": "MenuBook",
          "url": "https://docs.sspcloud.fr"
        }
      SOCIAL_MEDIA_TITLE: "SSPCloud Datalab"
      SOCIAL_MEDIA_DESCRIPTION: "Open Innovation Platform powered by Onyxia"
      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/custom-resources/social-preview.png"
      HEADER_TEXT_BOLD: "SSPCloud"
      HEADER_TEXT_FOCUS: "Datalab"
      FONT: |
        {
          fontFamily: "Geist",
          dirUrl: "%PUBLIC_URL%/fonts/Geist",
          "400": "Geist-Regular.woff2",
          "500": "Geist-Medium.woff2",
          "600": "Geist-SemiBold.woff2",
          "700": "Geist-Bold.woff2"
        }
      PALETTE_OVERRIDE_LIGHT: |
        {
            focus: {
                main: "#3B82F6",
                light: "#3B82F6",
            },
            light: {
                main: "#FAFAFA",
                light: "#FFFFFF",
                greyVariant1: "#EBEFF6"
            },
        }
      PALETTE_OVERRIDE_DARK: |
        {
            focus: {
              main: "#5695FB",
              light: "#5695FB",
            },
            dark: {
              main: "#0A152B",
              light: "#040B17",
            },
        }
      HOMEPAGE_MAIN_ASSET: "false"
      CUSTOM_HTML_HEAD: |
          <link rel="stylesheet" href="%PUBLIC_URL%/custom-resources/main.css"></link>
      BACKGROUND_ASSET: |
        {
          "light": "%PUBLIC_URL%/custom-resources/OnyxiaNeumorphismLightMode.svg",
          "dark": "%PUBLIC_URL%/custom-resources/OnyxiaNeumorphismDarkMode.svg"
        }
      CONTACT_FOR_ADDING_EMAIL_DOMAIN: |
        {
          "en": "If your email domain is not yet allowed [contact us](https://3innovation.slack.com/signup#/domain-signup)",
          "fr": "Si votre domaine de messagerie n'est pas encore autorisé [contactez-nous](https://3innovation.slack.com/signup#/domain-signup)"
        }
```
{% endcode %}
{% endtab %}

{% tab title="France" %}
<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption><p>Light Mode</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption><p>Dark Mode</p></figcaption></figure>

{% code title="onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      #ONYXIA_API_URL: https://datalab.sspcloud.fr/api
      CUSTOM_RESOURCES: "https://www.sspcloud.fr/france/custom-resources.zip"
      FONT: |
        { 
          fontFamily: "Marianne", 
          dirUrl: "%PUBLIC_URL%/custom-resources/fonts/Marianne", 
          "400": "Marianne-Regular.woff2",
          "400-italic": "Marianne-Regular_Italic.woff2",
          "500": "Marianne-Medium.woff2",
          "700": "Marianne-Bold.woff2",
          "700-italic": "Marianne-Bold_Italic.woff2"
        }
      PALETTE_OVERRIDE: |
        {
          focus: {
            main: "#000091",
            light: "#9A9AFF",
            light2: "#E5E5F4"
          },
          dark: {
            main: "#2A2A2A",
            light: "#383838",
            greyVariant1: "#161616",
            greyVariant2: "#9C9C9C",
            greyVariant3: "#CECECE",
            greyVariant4: "#E5E5E5"
          },
          light: {
            main: "#F1F0EB",
            light: "#FDFDFC",
            greyVariant1: "#E6E6E6",
            greyVariant2: "#C9C9C9",
            greyVariant3: "#9E9E9E",
            greyVariant4: "#747474"
          }
        }
      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/custom-resources/preview-france.png"
      HOMEPAGE_MAIN_ASSET: "false"
```
{% endcode %}
{% endtab %}

{% tab title="Honey" %}
<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption><p>Light mode</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption><p>Dark mode</p></figcaption></figure>

{% code title="onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      #ONYXIA_API_URL: https://datalab.sspcloud.fr/api
      CUSTOM_RESOURCES: "https://www.sspcloud.fr/honey/custom-resources.zip"
      HEADER_LOGO: "%PUBLIC_URL%/custom-resources/dapla_honey.svg"
      HEADER_TEXT_BOLD: "Onyxia Preview"
      HEADER_TEXT_FOCUS: "v10"
      HEADER_LINKS: |
        [
          {
            "label": "AIML4OS",
            "icon": "VideoCall",
            "url": "https://insee-fr.zoom.us/webinar/register/WN_6dhMgoUvRXmkiyvYYIyKvw"
          }
        ]
      PALETTE_OVERRIDE: |
        {
          "focus": {
            "main": "#FF9100", // Light mode focus
            light: "#FAB900", // Dark mode focus
          },
          "limeGreen": {
              "main": "#00DF0A"
          },
          "dapla": {
            yellow: "#FAB900",
            darkerYellow: "#FF9100"
          }
        }
      FONT: |
        {
          fontFamily: "Geist",
          dirUrl: "%PUBLIC_URL%/custom-resources/fonts/Geist",
          "400": "Geist-Regular.woff2",
          "500": "Geist-Medium.woff2",
          "600": "Geist-SemiBold.woff2",
          "700": "Geist-Bold.woff2"
        }
      HOMEPAGE_MAIN_ASSET: "%PUBLIC_URL%/custom-resources/dapla_bee_logo.png"
      HOMEPAGE_MAIN_ASSET_SCALE_FACTOR: "0.8"
      HOMEPAGE_MAIN_ASSET_Y_OFFSET: "3rem"
      HEADER_HIDE_ONYXIA: "true"
      BACKGROUND_ASSET: |
        {
          dark: "%PUBLIC_URL%/custom-resources/dapla_background_dark.svg",
          light: "%PUBLIC_URL%/custom-resources/dapla_background_light.svg"
        }
      #HOMEPAGE_CARDS: "[]"
      ENABLED_LANGUAGES: "no,en"
      HOMEPAGE_HERO_TEXT: |
        {
          en: "Welcome to the Dapla **Lab**",
          no: "Velkommen til Dapla **Lab**",
        }
      HOMEPAGE_HERO_TEXT_AUTHENTICATED: |
        {
          en: "Welcome %USER_FIRSTNAME%!",
          no: "Velkommen %USER_FIRSTNAME%!",
        }
      TERMS_OF_SERVICES: |
        {
          en: "%PUBLIC_URL%/custom-resources/tos_en.md",
          fr: "%PUBLIC_URL%/custom-resources/tos_fr.md",
        }
      CUSTOM_HTML_HEAD: |
        <link rel="stylesheet" href="%PUBLIC_URL%/custom-resources/custom.css">
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Additional Notes

Note that your custom assets are imported into your Onyxia instance via the use of the `CUSTOM_RESOURCES` parameter, url of a ZIP archive that should contain your assets.

{% hint style="info" %}
Onyxia is configured to make the the browser cache assets so they are not re-downloaded each time the user access the app.

If you update some of your asset but keep the same URL, you can force the browser of your users to download the new version by adding a query parameter to the URL. Eample:

`HEADER_LOGO: "%PUBLIC_URL%/custom-resources/logo.svg?v=2"`
{% endhint %}

Make sure to checkout the version of this document that matches the Onyxia version that you are deploying. [See releases](https://github.com/InseeFrLab/onyxia/releases).
