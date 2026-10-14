# 🔼 v10 -> v11

{% hint style="danger" %}
**Breaking changes**

* `workingDirectory` has been removed.
* `bookmarkedDirectories` has been renamed to `bookmarks`.
* `S3.sts.role` is now required.
{% endhint %}

v11 reworks the S3 configuration model in Onyxia.

Release notes:

{% embed url="https://datalab.sspcloud.fr/document?source=%2522%252Fcustom-resources%252Frelease-note-s3-explorer%252Fen.md%2522" %}

For the new S3 configuration model, see [the S3 configuration reference](https://app.gitbook.com/s/wJmp0a47ql4UqeqjlOOk/admin-doc/s3-configuration).

## Migration guide

### Replace `workingDirectory` with `bookmarks`

`workingDirectory` no longer exists in v11.

You now define explicit S3 `bookmarks` inside each S3 profile. Each bookmark points to a bucket or prefix with `s3Uri`. When you use `$1`, Onyxia substitutes it with the value of `claimName`.

{% tabs %}
{% tab title="`bucketMode: multi`" %}
{% code title="apps/onyxia/values.yaml" %}
```diff
 onyxia:
   api:
     regions:
       [
         {
           "data": {
             "S3": {
-              "workingDirectory": {
-                "bucketMode": "multi",
-                "bucketNamePrefix": "",
-                "bucketNamePrefixGroup": "projet-"
-              },
+              "bookmarks": [
+                {
+                  "s3Uri": "s3://$1/",
+                  "title": "Personal",
+                  "claimName": "preferred_username"
+                },
+                {
+                  "s3Uri": "s3://projet-$1/",
+                  "title": "Group $1",
+                  "claimName": "groups"
+                }
+              ]
             }
           }
         }
       ]
```
{% endcode %}
{% endtab %}

{% tab title="`bucketMode: shared`" %}
{% code title="apps/onyxia/values.yaml" %}
```diff
 onyxia:
   api:
     regions:
       [
         {
           "data": {
             "S3": {
-              "workingDirectory": {
-                "bucketMode": "shared",
-                "bucketName": "onyxia",
-                "prefix": "user-",
-                "prefixGroup": "project-"
-              }
+              "bookmarks": [
+                {
+                  "s3Uri": "s3://onyxia/user-$1/",
+                  "title": "Personal",
+                  "claimName": "preferred_username"
+                },
+                {
+                  "s3Uri": "s3://onyxia/project-$1/",
+                  "title": "Group $1",
+                  "claimName": "groups"
+                }
+              ]
             }
           }
         }
       ]
```
{% endcode %}
{% endtab %}
{% endtabs %}

Use a user claim such as `preferred_username` for personal buckets. Use a group claim such as `groups` for shared buckets.

### Rename `bookmarkedDirectories` to `bookmarks`

Existing saved locations must use the new key and the new path format.

{% code title="apps/onyxia/values.yaml" %}
```diff
 onyxia:
   api:
     regions:
       [
         {
           "data": {
             "S3": {
-              "bookmarkedDirectories": [
+              "bookmarks": [
                 {
-                  "fullPath": "donnees-insee/diffusion/",
+                  "s3Uri": "s3://donnees-insee/diffusion/",
                   "title": {
                     "fr": "Données de diffusion",
                     "en": "Dissemination Data"
                   }
                 }
               ]
             }
           }
         }
       ]
```
{% endcode %}

If you used extra bookmark metadata, verify it against the new S3 schema.

### Add `sts.role`

`sts.role` is now mandatory.

This also applies to MinIO setups where `roleARN` and `roleSessionName` were previously omitted. In most cases, set `profileName` to `default` and keep the other two fields empty.

{% code title="apps/onyxia/values.yaml" %}
```diff
 onyxia:
   api:
     regions: [
       {
         data: {
           S3: {
             URL: "https://minio.lab.my-domain.net",
             sts: {
+              role: {
+                profileName: "default",
+                roleARN: "",
+                roleSessionName: ""
+              }
             }
           }
         }
       }
     ]
```
{% endcode %}

{% hint style="info" %}
After the migration, users navigate S3 through the bookmarks you define in each profile.
{% endhint %}
