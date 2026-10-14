# March 2025 community call

Community call 03/28/2025

Project's news :

* New repo ! Awesome Onyxia : [https://github.com/onyxia-datalab/awesome-onyxia](https://github.com/onyxia-datalab/awesome-onyxia) Listing of resources related to the Onyxia ecosystem. Feel free to contribute :)
* Onboarding module in go : still WIP, not much this month. Contributions welcome, please join #dev-rewrite-to-go on Slack
* Feature request : Customizable User Profile. [https://github.com/InseeFrLab/onyxia/discussions/954](https://github.com/InseeFrLab/onyxia/discussions/954) . Work currently in progress to implement this using json schemas. Admin of the instance would specify a json schema describing the user profile. UI would render it and let user customize their profile (e.g git configuration). All the data would be then available for injection in the service launcher form.
* Debate : overwriteSchemaWith / patchSchemaWith (@Gaspard) : [https://github.com/InseeFrLab/onyxia-api/pull/573](https://github.com/InseeFrLab/onyxia-api/pull/573) Currently Onyxia is relying on overwriteSchemaWith that only allow to replace a schema part while discarding the existing one. Patchschemawith would allow to keep the existing definition (e.g when using Onyxia's opensource catalogs) and upstream changes while patching only what's necessary. Would also reduce duplication.
* New customization features:
  * Different palette for the dark and light mode
  * Gradiant background color
* Desktop App:
  * It would be an electron wrapper around Onyxia Web
  * The main goal would be to be able to bypass CORS issues (that are a blocker for accessing public S3 Bucket through the Onyxia UI)
* New button for accessing Keycloak user profile (when applicable). Do you want an option to disable it?

Community discussions :

* SSB : plugin showcase :\
  [https://github.com/statisticsnorway/onyxia/tree/ssb-assets/web/public/custom-resources](https://github.com/statisticsnorway/onyxia/tree/ssb-assets/web/public/custom-resources)
