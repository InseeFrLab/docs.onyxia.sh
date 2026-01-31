# September 2025 community call

Community call 09/25/2025

## Onyxia news

* [CVE-2025-58366](https://github.com/InseeFrLab/onyxia/security/advisories/GHSA-m773-6vm8-8x6q) Private helm repository credentials leak : only affected if you were using private helm catalogs (specifying credentials in the `catalogs` json). Patched on Onyxia 10.28. Take a look at our "vulnerability disclosure" documentation page and feel free to register to our security mailing to be notified when a vulnerability is discovered : https://docs.onyxia.sh/vulnerability-disclosure
* working on data catolog https://github.com/InseeFrLab/onyxia/issues/1021
* Reminder : the onboarding module, rewritten in go, is up for testing and use. It has been in use in production at SSPCloud for a month now. To test it, set `onboarding.enabled=true` in your chart values https://github.com/InseeFrLab/onyxia/blob/eeb00ae9d7047849b06dd5244eb1c4a4806db4ae/helm-chart/values.yaml#L317 no additional change is needed, we aimed for fully compatibility with the existing onboarding behaviour. Feedback welcome ! Onboarding code is hosted on the `onyxia-datalab` org on github : https://github.com/onyxia-datalab/onyxia-onboarding
* More work in progress in the go migration / rewrite, now working on the "main" API (services, my-lab ...) and moving onto a monorepo for all backend modules (onboarding, services ...) : https://github.com/onyxia-datalab/onyxia-backend Feel free to join the team :)





