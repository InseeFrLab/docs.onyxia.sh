---
description: >-
  Customize your Onyxia instance with your assets and your colors, make it your
  own!
---

# 🎨 Theme and branding

{% hint style="info" %}
A video tutoriel is in production
{% endhint %}

The full documentation of the avalable parameter can be found here:

{% embed url="https://github.com/InseeFrLab/onyxia/blob/main/web/.env" %}

Note that your custom assets are imported into your Onyxia instance via the use of the `CUSTOM_RESOURCES` parameter, url of a ZIP archive that should contain you assets. An example is given at the top of the [`.env`](https://github.com/InseeFrLab/onyxia/blob/main/web/.env) file.&#x20;

{% hint style="info" %}
Onyxia is configured to make the the browser cache assets so they are not re-downloaded each time the user access the app.&#x20;

If you update some of your asset but keep the same URL, you can force the browser of your users to download the new version by adding a query parameter to the URL. Eample:&#x20;

`HEADER_LOGO: "%PUBLIC_URL%/custom-resources/logo.svg?v=2"`
{% endhint %}

Make sure to checkout the version of this document that matches the Onyxia version that you are deploying. [See releases](https://github.com/InseeFrLab/onyxia/releases).

## Default looks

Here are two base look that you can use a starting point of your configuration.

### France

👉 [Theme preview](https://datalab.sspcloud.fr/?FONT=%7B%20%0A%20%20fontFamily%3A%20%22Marianne%22%2C%20%0A%20%20dirUrl%3A%20%22%25PUBLIC\_URL%25%2Ffonts%2FMarianne%22%2C%20%0A%20%20%22400%22%3A%20%22Marianne-Regular.woff2%22%2C%0A%20%20%22400-italic%22%3A%20%22Marianne-Regular\_Italic.woff2%22%2C%0A%20%20%22500%22%3A%20%22Marianne-Medium.woff2%22%2C%0A%20%20%22700%22%3A%20%22Marianne-Bold.woff2%22%2C%0A%20%20%22700-italic%22%3A%20%22Marianne-Bold\_Italic.woff2%22%0A%7D%0A\&PALETTE\_OVERRIDE=%7B%0A%20%20focus%3A%20%7B%0A%20%20%20%20main%3A%20%22%23000091%22%2C%0A%20%20%20%20light%3A%20%22%239A9AFF%22%2C%0A%20%20%20%20light2%3A%20%22%23E5E5F4%22%0A%20%20%7D%2C%0A%20%20dark%3A%20%7B%0A%20%20%20%20main%3A%20%22%232A2A2A%22%2C%0A%20%20%20%20light%3A%20%22%23383838%22%2C%0A%20%20%20%20greyVariant1%3A%20%22%23161616%22%2C%0A%20%20%20%20greyVariant2%3A%20%22%239C9C9C%22%2C%0A%20%20%20%20greyVariant3%3A%20%22%23CECECE%22%2C%0A%20%20%20%20greyVariant4%3A%20%22%23E5E5E5%22%0A%20%20%7D%2C%0A%20%20light%3A%20%7B%0A%20%20%20%20main%3A%20%22%23F1F0EB%22%2C%0A%20%20%20%20light%3A%20%22%23FDFDFC%22%2C%0A%20%20%20%20greyVariant1%3A%20%22%23E6E6E6%22%2C%0A%20%20%20%20greyVariant2%3A%20%22%23C9C9C9%22%2C%0A%20%20%20%20greyVariant3%3A%20%22%239E9E9E%22%2C%0A%20%20%20%20greyVariant4%3A%20%22%23747474%22%0A%20%20%7D%0A%7D%0A)

```yaml
  web:
    env:
      FONT: |
        { 
          fontFamily: "Marianne", 
          dirUrl: "%PUBLIC_URL%/fonts/Marianne", 
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
      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/preview-france.png"
      HOMEPAGE_MAIN_ASSET: "false"
```

### Ultraviolet

👉 [Theme preview](https://datalab.sspcloud.fr/?FONT=%7B%20%0A%20%20fontFamily%3A%20%22Geist%22%2C%20%0A%20%20dirUrl%3A%20%22%25PUBLIC\_URL%25%2Ffonts%2FGeist%22%2C%20%0A%20%20%22400%22%3A%20%22Geist-Regular.woff2%22%2C%0A%20%20%22500%22%3A%20%22Geist-Medium.woff2%22%2C%0A%20%20%22600%22%3A%20%22Geist-SemiBold.woff2%22%2C%0A%20%20%22700%22%3A%20%22Geist-Bold.woff2%22%0A%7D%0A\&PALETTE\_OVERRIDE=%7B%0A%20%20focus%3A%20%7B%0A%20%20%20%20main%3A%20%22%23067A76%22%2C%0A%20%20%20%20light%3A%20%22%230AD6CF%22%2C%0A%20%20%20%20light2%3A%20%22%23AEE4E3%22%0A%20%20%7D%2C%0A%20%20dark%3A%20%7B%0A%20%20%20%20main%3A%20%22%232D1C3A%22%2C%0A%20%20%20%20light%3A%20%22%234A3957%22%2C%0A%20%20%20%20greyVariant1%3A%20%22%2322122E%22%2C%0A%20%20%20%20greyVariant2%3A%20%22%23493E51%22%2C%0A%20%20%20%20greyVariant3%3A%20%22%23918A98%22%2C%0A%20%20%20%20greyVariant4%3A%20%22%23C0B8C6%22%0A%20%20%7D%2C%0A%20%20light%3A%20%7B%0A%20%20%20%20main%3A%20%22%23F7F5F4%22%2C%0A%20%20%20%20light%3A%20%22%23FDFDFC%22%2C%0A%20%20%20%20greyVariant1%3A%20%22%23E6E6E6%22%2C%0A%20%20%20%20greyVariant2%3A%20%22%23C9C9C9%22%2C%0A%20%20%20%20greyVariant3%3A%20%22%239E9E9E%22%2C%0A%20%20%20%20greyVariant4%3A%20%22%23747474%22%0A%20%20%7D%0A%7D%0A)

```yaml
web:
    env:
      FONT: |
        { 
          fontFamily: "Geist", 
          dirUrl: "%PUBLIC_URL%/fonts/Geist", 
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
      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/preview-ultraviolet.png"
```
