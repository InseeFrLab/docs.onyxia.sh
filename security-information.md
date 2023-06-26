---
description: Information about security considerations
---

# Security information

There are some sensitives features that you need to understand before installing Onyxia&#x20;

* Autolaunch feature

With this feature, you can create some http link which deploy an environment. It's very helpfull to start trainings easily. [You should consider to disable that because it can be dangerous for the user](https://github.com/InseeFrLab/onyxia-web/issues/548).

* Group feature

Onyxia is mainly design to give a user some resources to work : a namespace and a S3 bucket. There is also a feature to give multiple user access to the same resources (a project). It can be used by a malicious user in this group to use privilege of another member of this project
