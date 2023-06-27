---
description: Information about security considerations
---

# 🔓 Securing Onyxia Installation

#### 1. Autolaunch Feature

The autolaunch feature empowers you to create HTTP links that automatically deploy an environment. This is an invaluable tool for initiating trainings effortlessly. However, exercise caution while using it as it could pose a security risk to the user. Consider disabling this feature if it doesn't suit your requirements or if security is a primary concern.

#### 2. Group Feature

Onyxia is primarily designed to allocate resources such as a namespace and an S3 bucket to an individual user for work purposes. Additionally, it incorporates a feature that allows multiple users to share access to the same resources within a project. While this can be extremely beneficial for collaboration, be aware that it might be exploited by a malicious user within the group to leverage the privileges of another project member. Always monitor shared resources and maintain proper user access control to prevent such security breaches.
