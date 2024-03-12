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

### Deploying Keycloak

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
   4. On the tab **Themes**
      1. _Login theme_: **onyxia-web** (you can also select the login theme on a per client basis)
      2. _Email theme_: **onyxia-web**
   5. On the tab **Localization**
      1. _Internationalization_: **Enabled**
      2. _Supported locales_: \<Select the languages you wish to support>
   6. On the tab **Session**.
      1. SSO Session Idle: [14 days](#user-content-fn-2)[^2]
      2. SSO Session Max: [14 days](#user-content-fn-3)[^3]
      3. SSO Session Idle Remember Me: [14 days](#user-content-fn-4)[^4]
      4. SSO Session Max Remember Me: 14 days
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
...
```

Now our Keycloak server is fully configured we just need to update our Onyxia deployment to let it know about it.

### Updating the Onyxia configuration

In your GitOps repo you now want to update your onyxia configuration. &#x20;

```bash
git clone https://github.com/<your-github-org>/onyxia-ops
cd onyxia-ops
cd apps/onyxia
mv values-keycloak-enabled.yaml values.yaml
git commit -am "Enable keycloak"
git push
```

Here is the DIFF of the onyxia configuration: &#x20;

{% embed url="https://github.com/InseeFrLab/onyxia-ops/commit/37faa6390c9bc8c1efddfd3488dc06b38427b424" %}

Now your users should be able to create account, log-in, and start services on their own Kubernetes namespace.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption><p>The screen you shoud see when clicking on "login" in your Onyxia deployment</p></figcaption></figure>

Next step in the installation proccess it to enable all the S3 related features of Onyxia: &#x20;

{% content-ref url="data-s3.md" %}
[data-s3.md](data-s3.md)
{% endcontent-ref %}

[^1]: Search/replace CHANGEME

[^2]: Here, the approach depends on the security policy you wish to implement. If you prefer requiring users to log in again each time they navigate to your Onyxia instance, consider setting a shorter duration, such as 30 minutes. Be aware that setting it to 30 minutes means users will be automatically logged out if they remain inactive within the app for this period.  \
    [https://github.com/InseeFrLab/onyxia/assets/6702424/343f74e1-1f08-43e3-8a1d-ce92f8dedc2c\
    ](https://github.com/InseeFrLab/onyxia/assets/6702424/343f74e1-1f08-43e3-8a1d-ce92f8dedc2c)

[^3]: You'll likely want to set this value high. It determines the maximum duration for a continuously active user session. If a user is logged in and actively using the app, there's no need to disconnect them. &#x20;

[^4]: Modify this setting if you wish to apply a different policy for users logging in with the "remember me" option selected. In "remember me" mode, users can close their browser completely and will not need to log in again on their next visit, provided the session has not expired.  \
    [https://github.com/InseeFrLab/onyxia/assets/6702424/93b139cf-b0e7-4e4b-9811-bf9a9deaf144](https://github.com/InseeFrLab/onyxia/assets/6702424/93b139cf-b0e7-4e4b-9811-bf9a9deaf144)
