# April 2025 community call

Community call 04/24/2025

Project's news :

* New schedule for community calls ! Last thursday of the month at 16:30 Paris time. ICS calendar available : [https://docs.onyxia.sh/contributors-doc/community-calls](https://docs.onyxia.sh/contributors-doc/community-calls)
* Various improvements to My files
* WIP : Multiple STS configuration support. Still needs work especially on User interface and catalogs configuration / injection. Need to figure out what to do with the dropdown menu that currently allows switching between S3 configurations. Almost all the services (python, R ...) are ready to support multiple STS configurations but duckdb is not, an issue is currently open on their side.
* Work has started on "User profile" feature ([https://github.com/InseeFrLab/onyxia/discussions/954](https://github.com/InseeFrLab/onyxia/discussions/954)). Not testable yet but feedback welcome on usecase, would you use it ?
* Onyxia support for charts without values.schema.json by fallbacking into the the new YAML editor.
  * Possibility to see all defaults in the text editor, including the ones defined in the values.yaml.
* Customization: Possibility to define different color palettes for dark and light mode. Possibility to add custom CSS for light and dark mode.

Community discussions :

* Data(S3) configuration should allow configurations that don't contain prefix/ prefixGroup or bucketNamePrefix/ bucketNamePrefixGroup. Basically keeping S3 configuration / STS / injection but disabling user bucket / working directory path … In this setup, users will then run things like `mc ls s3` and have access to mulitple buckets not tied to their username.
* Document Group Projects : documentation is lackluster on how to configure / enable groups feature and what the feature is about (what the group gives access to, how it's supposed to be used …)
