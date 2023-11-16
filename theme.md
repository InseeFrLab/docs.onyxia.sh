---
description: >-
  Customize your Onyxia instance with your assets and your colors, make it your
  own!
---

# 🎨 Theme and branding

{% hint style="info" %}
A video tutoriel is in production&#x20;
{% endhint %}

The full documentation of the avalable parameter can be found here: &#x20;

{% embed url="https://github.com/InseeFrLab/onyxia/blob/main/web/.env" %}

Make sure to checkout the version of this document that matches the Onyxia version that you are deploying. [See releases](https://github.com/InseeFrLab/onyxia/releases).

## Default looks

Here are two base look that you can use a starting point of your configuration. &#x20;

### France

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

👉 [Theme preview](https://datalab.sspcloud.fr/?PALETTE\_OVERRIDE=%7B%22focus%22%3A%7B%22main%22%3A%22%23067A76%22%2C%22light%22%3A%22%230AD6CF%22%2C%22light2%22%3A%22%23AEE4E3%22%7D%2C%22dark%22%3A%7B%22main%22%3A%22%232D1C3A%22%2C%22light%22%3A%22%234A3957%22%2C%22greyVariant1%22%3A%22%2322122E%22%2C%22greyVariant2%22%3A%22%23493E51%22%2C%22greyVariant3%22%3A%22%23918A98%22%2C%22greyVariant4%22%3A%22%23C0B8C6%22%7D%2C%22light%22%3A%7B%22main%22%3A%22%23F7F5F4%22%2C%22light%22%3A%22%23FDFDFC%22%2C%22greyVariant1%22%3A%22%23E6E6E6%22%2C%22greyVariant2%22%3A%22%23C9C9C9%22%2C%22greyVariant3%22%3A%22%239E9E9E%22%2C%22greyVariant4%22%3A%22%23747474%22%7D%7D\&FONT=%7B%22fontFamily%22%3A%22Geist%22%2C%22dirUrl%22%3A%22%25PUBLIC\_URL%25%2Ffonts%2FGeist%22%2C%22400%22%3A%22Geist-Regular.woff2%22%2C%22500%22%3A%22Geist-Medium.woff2%22%2C%22600%22%3A%22Geist-SemiBold.woff2%22%2C%22700%22%3A%22Geist-Bold.woff2%22%7D)

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
      HOMEPAGE_MAIN_ASSET: "%PUBLIC_URL%/illustrations/projets.png"
      HOMEPAGE_MAIN_ASSET_SCALE_FACTOR: "1.5"
      HOMEPAGE_MAIN_ASSET_Y_OFFSET: "-50px"
      HOMEPAGE_MAIN_ASSET_X_OFFSET: "430px"
```
