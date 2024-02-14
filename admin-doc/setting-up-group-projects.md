---
description: >-
  Enabling a group of user to share the same Kubernetes namespace to work on
  something together.
---

# 👥 Setting up group projects

The user interface of onyxia enable to create project for groups of Onyxia user. &#x20;

Users will be able to dynamically switch from one project to another using a select input in the header.

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This select doesen't appear when the user isn't in any group project. &#x20;

All users of a group project shares:

* The Kubernetes namespace, in "My Services" you can see everything that's running, including services launched by other personne of the group. &#x20;
* Project settings. If a user change a project setting it affects every member of the group.
* Secrets
* S3 Bucket (or an S3 subpath)

As of today new group can only be created by Onyxia instance administrator, on demand and the procedure to create group is not publicly documented yet because we're still actively working on it.  \
However, if you want to enable this feature for your users, reach us, we will guide you through it! &#x20;

{% embed url="https://join.slack.com/t/3innovation/shared_invite/zt-1hnzukjcn-6biCSmVy4qvyDGwbNI~sWg" %}
