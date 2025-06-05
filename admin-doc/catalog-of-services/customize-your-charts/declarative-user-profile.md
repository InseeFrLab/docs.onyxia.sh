---
icon: user
---

# Declarative User Profile

You can define a custom user profile form that appears directly within the user interface.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Custom form defined by the Onyxia instance administrator</p></figcaption></figure>

This form is configured using a JSON Schema provided via your Onyxia `values.yaml`. Here's an example that produces the form shown above:

{% code title="onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    userProfile:
      enabled: true
      default:
        profileSchema: |
          {
            "type": "object",
            "properties": {
              "generalInfo": {
                "type": "object",
                "description": "General profile information",
                "properties": {
                  "firstName": {
                    "type": "string",
                    "title": "First name",
                    "description": "Your first name",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.given_name}}"
                    }
                  },
                  "familyName": {
                    "type": "string",
                    "title": "Family name",
                    "description": "Your family name",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.family_name}}"
                    }
                  },
                  "email": {
                    "type": "string",
                    "title": "Email",
                    "description": "Your email address",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.email}}"
                    }
                  }
                }
              },
              "git": {
                "type": "object",
                "description": "Git configuration",
                "properties": {
                  "username": {
                    "type": "string",
                    "title": "Git username",
                    "description": "Your username for Git operations (e.g. git commit, git push)",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{git.name}}"
                    }
                  },
                  "email": {
                    "type": "string",
                    "title": "Git email",
                    "description": "Your email for Git operations",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{git.email}}"
                    }
                  }
                }
              }
            }
          }
      roles:
        # NOTE: You can define role-specific schemas if needed.
        #- roleName: datascientist
        #  profileSchema: |
        #    ...
```
{% endcode %}

***

## Why Use a Custom User Profile?

Once defined, this form allows users to fill in personal and development-related information. These values become programmatically accessible, enabling dynamic behavior within your charts and deployments.

For example, with the schema above, and assuming the user has filled out the form as shown in the screenshot, the following values will be available in the Onyxia context:

{% code title="xOnyxiaContext.user.profile" %}
```json
{
  "generalInfo": {
    "firstName": "Joseph",
    "lastName": "Garrone",
    "email": "joseph.garrone@code.gouv.fr"
  },
  "git": {
    "username": "garronej",
    "email": "joseph.garrone.gj@gmail.com"
  }
}
```
{% endcode %}

These values can be injected into Helm charts. For instance:

```json
"x-onyxia": {
  "overwriteDefaultWith": "{{user.profile.generalInfo.lastName}}"
}
```

This will auto-fill the corresponding field with `"Garrone"`.

{% hint style="warning" %}
Each time you update the JSON Schema you provide to define the user profile, all existing values that the user might have filled will be lost. &#x20;
{% endhint %}

***

## Recap

* Define your schema in `onyxia.values.yaml`.
* Enable role-based customization if needed.
* Use the collected values in your Helm charts for a tailored, user-aware deployment experience.
