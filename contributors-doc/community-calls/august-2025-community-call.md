# August 2025 community call

Community call 08/28/2025

## Project news

* Release v10.27
  * Onboarding as a separate go module (test it, `onboarding.enabled=true` in your values : https://github.com/InseeFrLab/onyxia/blob/ba06172cba57b6893d0646f6521a1895532d844a/helm-chart/values.yaml#L316)
* State of go :
  * Onboarding is functional and at (almost, mainly missing events support) parity with current API
  * Work is now on `services` API
  * Code is available on a monorepo : https://github.com/onyxia-datalab/onyxia-backend
  * Feedback and contributions welcome ! #dev-rewrite-to-go on slack to discuss
* Bitnamigate happening today :scream: : https://github.com/bitnami/charts/issues/35164
  * Bitnami dropping / reducing docker & helm charts availability and support
  * Inseefrlab (defaults for Onyxia) catalogs have been updated this week to use `bitnamilegacy` (mainly for databases catalog)
  * Future : we will try to remove catalog dependencies to bitnami wherever and whenever it's feasible





