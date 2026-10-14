# 🔼 v9 -> v10

:warning: The quotas configuration has finally been cleaned up ! The "old style quota" that was deprecated for more than 1 year has finally been removed. **Don't worry, migration is easy : instead of using enabled and default, you have to enable and define quotas for user and groups separately**. See [https://github.com/InseeFrLab/onyxia-api/blob/main/docs/region-configuration.md#quotas-properties](https://github.com/InseeFrLab/onyxia-api/blob/main/docs/region-configuration.md#quotas-properties)  for details. Note that you can now define quotas per role thanks to the new role system introduced in this release.  :warning:

:warning: This release has refactored some endpoints of the API, especially endpoints regarding catalogs. \
**If you are relying on the Onyxia-API directly** (e.g you built a custom frontend) you will notice that some endpoints have been removed / renamed. Note that we don't recommend nor support relying on the Onyxia-API directly but if you do, we would love to hear from you and your usecase. Maybe it could be officially supported and benefit others. :warning:&#x20;

{% hint style="info" %}
**New feature**\
You can use overwrite schemas based on role. You can configure role inside Onyxia and provide schema for each role. For more details, refer to the documentation: [Onyxia v10 Catalog.](https://app.gitbook.com/s/x3LIftMZY501x5liXUPV/admin-doc/catalog-of-services)
{% endhint %}
