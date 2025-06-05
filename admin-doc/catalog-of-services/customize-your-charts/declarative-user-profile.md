---
icon: user
---

# Declarative User Profile

You can create a custom form that will appear in the user's profile section. &#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Custom form defined by the Onxia instance administrator</p></figcaption></figure>

To make this form appear, you must provide a JSON Schema in your Onyxia's values. Let's see a schema that generates the above form:

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
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.given_name}}"
                    },
                    "description": "Your first name"
                  },
                  "familyName": {
                    "type": "string",
                    "title": "Family name",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.family_name}}"
                    },
                    "description": "Your family name"
                  },
                  "email": {
                    "type": "string",
                    "title": "Email",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{user.decodedIdToken.email}}"
                    },
                    "description": "Your email address"
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
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{git.name}}"
                    },
                    "description": "Your username for git operations (e.g. git commit, git push)"
                  },
                  "email": {
                    "type": "string",
                    "title": "Git email",
                    "description": "Your username for git operations (e.g. git commit, git push)",
                    "x-onyxia": {
                      "overwriteDefaultWith": "{{git.email}}"
                    }
                  }
                }
              }
            }
          }
      roles:
        # NOTE: You can have different shema depending on the role that the use has.
        #- roleName: datascientist
        #  profileSchema: |
        #    ...
```
{% endcode %}

Now you might ask, what's the pupose of this form right?  \
Well you can use the values that the user might have populated in your custom chart.

In the XOnyxiaContext, with the above JSON Shema configured, and the user having filled the form as in the first screenshot you will get:

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

This means you can have a chart that defines:

```json
"x-onyxia": {
    "overwriteDefaultWith": "{{user.profile.generalInfo.lastName}}"
},
```

And the value will be auto filled by "Garrone".
