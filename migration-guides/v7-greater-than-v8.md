# ⬆ v7->v8

In this major release, the main change is about how Onyxia uses S3 storage.&#x20;

<pre class="language-diff" data-title="values.yaml"><code class="lang-diff"><strong>    regions:
</strong>      [
          ...
          "data": {
-               "S3":
-                {
-                  "type": "minio",
-                  "URL": "https://minio.example.org",
-                  "region": "us-east-1",
-                  "bucketPrefix": "user-",
-                  "groupBucketPrefix": "projet-",
-                  "bucketClaim": "preferred_username",
-                  "defaultDurationSeconds": 86400,
-                  "oidcConfiguration":
-                    {
-                      "issuerURI": "https://auth.example.org/auth/realms/myrealm",
-                      "clientID": "minio",
-                    },
-                },
+              "S3":
+                {
+                  "URL": "https://minio.example.org",
+                  "region": "us-east-1",
+                  "pathStyleAccess": "true",
                   STS is no more mandatory. Onyxia can wait keys provided by the users. 
                   It's obviously less easy to use. Use STS if possible. 
+                  "sts": {
+                      "durationSeconds": 86400,
+                      "oidcConfiguration":
+                      {
+                        "issuerURI": "https://auth.example.org/auth/realms/myrealm",
+                        "clientID": "minio",
+                      },
+                  },
                   workingDirectory specify the default storage directory for your user.
                   There are two ways to use it: 'multi' where each user has their own buckets, and 'shared' where users just have a subpath.
                   two examples :                  
+                  "workingDirectory": {
+                      "bucketMode": "multi",
+                      "bucketNamePrefix": "user-",
+                      "bucketNamePrefixGroup": "projet-",
+                  },
+                  "workingDirectory": {
+                      "bucketMode": "shared",
+                      "bucketName": "onyxia",
+                      "prefix": "user-",
+                      "prefixGroup": "projet-",
+                  },
+                },
            },
</code></pre>
