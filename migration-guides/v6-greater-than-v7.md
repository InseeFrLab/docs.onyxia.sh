# ⬆ v6 -> v7

In this major version a lot of the parameters of the webapp have been updated/refined.\
Here is the changes you need to apply to your values.json to migrate smoothly.

## The `THEME_ID` parameter has been removed.

Onyxia is now fully customizable instead of just letting you pick within a handful of predefined themes.

### If you where using the `france` theme:

{% code title="values.yaml" %}
```diff
 onyxia:
   web:
     env:
-      THEME_ID: france
+      FONT: |
+        { 
+          fontFamily: "Marianne", 
+          dirUrl: "%PUBLIC_URL%/fonts/Marianne", 
+          "400": "Marianne-Regular.woff2",
+          "400-italic": "Marianne-Regular_Italic.woff2",
+          "500": "Marianne-Medium.woff2",
+          "700": "Marianne-Bold.woff2",
+          "700-italic": "Marianne-Bold_Italic.woff2"
+        }
+      PALETTE_OVERRIDE: |
+        {
+          focus: {
+            main: "#000091",
+            light: "#9A9AFF",
+            light2: "#E5E5F4"
+          },
+          dark: {
+            main: "#2A2A2A",
+            light: "#383838",
+            greyVariant1: "#161616",
+            greyVariant2: "#9C9C9C",
+            greyVariant3: "#CECECE",
+            greyVariant4: "#E5E5E5"
+          },
+          light: {
+            main: "#F1F0EB",
+            light: "#FDFDFC",
+            greyVariant1: "#E6E6E6",
+            greyVariant2: "#C9C9C9",
+            greyVariant3: "#9E9E9E",
+            greyVariant4: "#747474"
+          }
+        }
+      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/preview-france.png"
```
{% endcode %}

### If you where using the `ultraviolet` theme:

{% code title="values.yaml" %}
```diff
 onyxia:
   web:
     env:
-      THEME_ID: ultraviolet
+      PALETTE_OVERRIDE: |
+        {
+          focus: {
+            main: "#067A76",
+            light: "#0AD6CF",
+            light2: "#AEE4E3"
+          },
+          dark: {
+            main: "#2D1C3A",
+            light: "#4A3957",
+            greyVariant1: "#22122E",
+            greyVariant2: "#493E51",
+            greyVariant3: "#918A98",
+            greyVariant4: "#C0B8C6"
+          },
+          light: {
+            main: "#F7F5F4",
+            light: "#FDFDFC",
+            greyVariant1: "#E6E6E6",
+            greyVariant2: "#C9C9C9",
+            greyVariant3: "#9E9E9E",
+            greyVariant4: "#747474"
+          }
+        }
+      SOCIAL_MEDIA_IMAGE: "%PUBLIC_URL%/preview-ultraviolet.png"
```
{% endcode %}

### If you where using the `verdant` theme:

{% code title="values.yaml" %}
```diff
 onyxia:
   web:
     env:
-      THEME_ID: verdant
+      PALETTE_OVERRIDE: |
+        focus: {
+            main: "#1F8D49",
+            light: "#4EFB8D",
+            light2: "#DFFEE6"
+        },
+        light: {
+            main: "#F4F6FF",
+            light: "#F6F6F6",
+            greyVariant1: "#E6E6E6",
+            greyVariant2: "#C9C9C9",
+            greyVariant3: "#9E9E9E",
+            greyVariant4: "#747474"
+        }
```
{% endcode %}

## Header parameters

{% code title="values.yaml" %}
```diff
 onyxia:
   web:
     env:
-      HEADER_ORGANIZATION: SSP Cloud
+      HEADER_TEXT_BOLD: SSP Cloud
-      HEADER_USECASE_DESCRIPTION: Datalab
+      HEADER_TEXT_FOCUS: Datalab
-      DESCRIPTION: Shared platform for statistical data processing and data science services
+      SOCIAL_MEDIA_DESCRIPTION: Shared platform for statistical data processing and data science services
+      SOCIAL_MEDIA_TITLE: Datalab - SSP Cloud
```
{% endcode %}

## Links in the header and the left bar

In addition to the parameter `EXTRA_LEFTBAR_ITEMS` having being renamed to `LEFTBAR_LINKS` the `iconId` property has been renamed `icon` and you can now use any icon from [the Material Design library](https://mui.com/material-ui/material-icons) or even provide your own icons.\
Please refer to [the new documentation of the `HEADER_LINKS` parameter](https://github.com/InseeFrLab/onyxia/blob/v7.0.0/web/.env).

{% code title="values.yaml" %}
```diff
 onyxia:
   web:
     env:
-      EXTRA_LEFTBAR_ITEMS: |
+      LEFTBAR_LINKS: |
```
{% endcode %}

### Assets must now be bundled

You must now bundle your assets such as the terms of services inside your onyxia instance. The newer version of Onyxia won't fetch resource from arbitrary URLs.  \
See `CUSTOM_RESOURCES` in [the .env file](https://github.com/InseeFrLab/onyxia/blob/main/web/.env).

### Keycloak Theme

If you are using the Onyxia Keycloak theme and your instance is public you might want to fill up the `ONYXIA_` prefixed environement variable in your Keycloak envs.  \
See [install doc](../#enabling-user-authentication).
