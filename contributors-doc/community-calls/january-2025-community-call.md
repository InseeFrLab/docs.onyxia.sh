# January 2025 community call

Community call 01/31/25

* CVE\
  A 9.4 vulnerability has been found (thanks team norway !) in Onyxia-API at the end of December.\
  Read more here : [https://nvd.nist.gov/vuln/detail/CVE-2024-56333](https://nvd.nist.gov/vuln/detail/CVE-2024-56333)\
  We created a new section on the docs for everything related to security including a mailing list : [https://docs.onyxia.sh/vulnerability-disclosure](https://docs.onyxia.sh/vulnerability-disclosure)
* API rewrite Java => Go\
  We started the process of rewriting the Onyxia-API that is currently written in Java to Golang as go is a lot more integrated with cloud native technologies. It would greatly improve performance, maintainability (letting us get rid of the Helm wrapper we built) and security.\
  We take this opportunity to also split the API into separated modules, starting with the Onboarding.\
  See discussion here : [https://github.com/InseeFrLab/onyxia/discussions/925](https://github.com/InseeFrLab/onyxia/discussions/925)\
  Feel free to join the effort by joining the #dev-rewrite-to-go channel on Slack\
  Also we created the [https://github.com/onyxia-datalab](https://github.com/onyxia-datalab) org
* YAML editor\
  New feature ! Making progress towards more support for advanced users such as developpers. Letting them directly modify values instead of using the UI form. Also support for charts that don't have a `values.schema.json`.
* Parquet\
  Improvements on how parquet are displayed in the Data explorer, also improvements on file format detection

Mercator :

* Updated their fork from v9 => v10. Quite some work was needed but happy with the new features.

API rewrite from Java to Go is not just a rewrite in another language, it's also an opportunity to change the architecture. Discussions on key points such as if / how we should support things other than helm packages are currently taking place so anyone interested in this are more than welcome to join the discussion #dev-rewrite-to-go and [https://github.com/InseeFrLab/onyxia/discussions/925](https://github.com/InseeFrLab/onyxia/discussions/925)
