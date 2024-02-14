---
description: Using Keycloak to enable user authentication
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# 🔑 User authentication

Let's setup Keycloak to enable users to create account and login to our Onyxia.

{% hint style="success" %}
Note that in this instalation guide we make you use Keycloak but you can use any identity server that is Open ID Connect compliant.
{% endhint %}

### Installing Keycloak on your cluster

We're going to install Keycloak just like we installed Onyxia. &#x20;

Before anything open [`apps/keycloak/values.yaml`](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/keycloak/values.yaml) in your onyxia-ops repo and [change the passwords](#user-content-fn-1)[^1]. Also write down the [`keycloak.auth.adminPassword`](https://github.com/InseeFrLab/onyxia-ops/blob/bad75636d72c20c48f1b34ec08593df83ee6c9a6/apps/keycloak/values.yaml#L11), you'll need it to connect to the Keycloak console. &#x20;

{% embed url="https://app.tango.us/app/embed/dbb21e90-db2c-41f4-b2ab-5f8b9f4d33c0" %}

{% hint style="info" %}
Try to remember, when you [update Onyxia in `apps/onyxia/Chart.yaml`](https://github.com/InseeFrLab/onyxia-ops/blob/bad75636d72c20c48f1b34ec08593df83ee6c9a6/apps/onyxia/Chart.yaml#L6) to also update [the Onyxia theme in `apps/keycloak/values.yaml`](https://github.com/InseeFrLab/onyxia-ops/blob/bad75636d72c20c48f1b34ec08593df83ee6c9a6/apps/keycloak/values.yaml#L69).
{% endhint %}

### Configuring Keycloak

You can now login to the **administration console** of **https://auth.lab.my-domain.net/auth/** and login using username: keycloak and password: \<the one you've wrote down earlier>.

1. Create a realm called "datalab" (or something else), go to **Realm settings**
   1. On the tab General
      1. _User Profile Enabled_: **On**
   2. On the tab **login**
      1. _User registration_: **On**
      2. _Forgot password_: **On**
      3. _Remember me_: **On**
   3. On the tab **email,** we give an example with [AWS SES](https://aws.amazon.com/ses/), if you don't have a SMTP server at hand you can skip this by going to **Authentication** (on the left panel) -> Tab **Required Actions** -> Uncheck "set as default action" **Verify Email**. Be aware that with email verification disable, anyone will be able to sign up to your service.
      1. _From_: **noreply@lab.my-domain.net**
      2. _Host_: **email-smtp.us-east-2.amazonaws.com**
      3. _Port_: **465**
      4. _Authentication_: **enabled**
      5. _Username_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\***
      6. _Password_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\***
      7. When clicking "save" you'll be asked for a test email, you have to provide one that correspond **to a pre-existing user** or you will get a silent error and the credentials won't be saved.
   4. On the tab Themes
      1. _Login theme_: **onyxia-web** (you can also select the login theme on a per client basis)
      2. _Email theme_: **onyxia-web**
   5. On the tab **Localization**
      1. _Internationalization_: **Enabled**
      2. _Supported locales_: \<Select the languages you wish to support>
   6. On the tab **Session**
      1. SSO Session Idle: 14 days - This setting and the following two are so that when the user click "remember me" when he logs in, he dosen't have to loggin again for the next two weeks.
      2. SSO Session Idle Remember Me: 14 days
      3. SSO Session Max Remember Me: 14 days
2. Create a client with client ID "onyxia"
   1. _Root URL_: **https://datalab.my-domain.net/**
   2. _Valid redirect URIs_: **https://datalab.my-domain.net/\***
   3. _Web origins_: **\***
   4. Login theme: **onyxia-web**
3. In **Authentication** (on the left panel) -> Tab **Required Actions** enable and set as default action **Therms and Conditions.**

Now you want to ensure that the username chosen by your users complies with Onyxia requirement (only alphanumerical characters) and define a list of email domain allowed to register to your service.

Go to **Realm Settings** (on the left panel) -> Tab **User Profile** (this tab shows up only if User Profile is enabled in the General tab and you can enable user profile only if you have started Keycloak with `-Dkeycloak.profile=preview)` -> **JSON Editor**.

Now you can edit the file as suggested in the following DIFF snippet. Be mindful that in this example we only allow emails @gmail.com and @hotmail.com to register you want to edit that.

```diff
{
  "attributes": [
    {
      "name": "username",
      "displayName": "${username}",
      "validations": {
        "length": {
          "min": 3,
          "max": 255
        },
+       "pattern": {
+         "error-message": "${alphanumericalCharsOnly}",
+         "pattern": "^[a-zA-Z0-9]*$"
+       },
        "username-prohibited-characters": {}
      }
    },
    {
      "name": "email",
      "displayName": "${email}",
      "validations": {
        "email": {},
+       "pattern": {
+         "pattern": "^[^@]+@([^.]+\\.)*((gmail\\.com)|(hotmail\\.com))$"
+       },
        "length": {
          "max": 255
        }
      }
    },
    {
      "name": "firstName",
      "displayName": "${firstName}",
      "required": {
        "roles": [
          "user"
        ]
      },
      "permissions": {
        "view": [
          "admin",
          "user"
        ],
        "edit": [
          "admin",
          "user"
        ]
      },
      "validations": {
        "length": {
          "max": 255
        },
        "person-name-prohibited-characters": {}
      }
    },
    {
      "name": "lastName",
      "displayName": "${lastName}",
      "required": {
        "roles": [
          "user"
        ]
      },
      "permissions": {
        "view": [
          "admin",
          "user"
        ],
        "edit": [
          "admin",
          "user"
        ]
      },
      "validations": {
        "length": {
          "max": 255
        },
        "person-name-prohibited-characters": {}
      }
    }
  ]
}
```

Now our Keycloak server is fully configured we just need to update our Onyxia deployment to let it know about it.

### Updating our Onyxia configuration

In your GitOps repo you now want to overwrite the content of apps/onyxia/values.yaml by apps/onyxia.values-keycloak-enabled.yaml.  (`mv onyxia.values-keycloak-enabled.yaml onyxia.values.yaml`)

{% embed url="https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/onyxia/values-keycloak-enabled.yaml" %}

Now your users should be able to create account, log-in, and start services on their own Kubernetes namespace.

Next step in the installation proccess it to enable all the S3 related features of Onyxia: &#x20;

{% content-ref url="data-s3.md" %}
[data-s3.md](data-s3.md)
{% endcontent-ref %}

[^1]: Search/replace CHANGEME
