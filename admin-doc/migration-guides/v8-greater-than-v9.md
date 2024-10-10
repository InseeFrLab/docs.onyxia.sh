# ⬆️ v8 -> v9

{% hint style="warning" %}
tl;dr: **Breaking change**, `defaultConfiguration` in region configuration is not allowed anymore and has been replaced by JSONSchemas override using the new `api.schemas` key from v9 helm chart.
{% endhint %}

Onyxia v9 allows administrators to define custom JSON schemas, allowing them to override the default schemas provided by the chart. Prior to this change, Onyxia relied on providing default values for specific keys in the region configuration : `defaultConfiguration`.&#x20;

Chart owners can now define which properties can be overridden using a JSON Schema.

Here is an example of a Chart that supports JSONSchemas (taken from the default IDE catalog, see [this link](https://github.com/InseeFrLab/helm-charts-interactive-services/blob/3f32bcd4fc16ee194782616d0d4634197ec75acb/charts/vscode-python/values.schema.json#L603C5-L612C6)) : &#x20;

{% code title="values.schema.json" %}
```json
"nodeSelector": {
      "type": "object",
      "description": "NodeSelector",
      "default": {},
      "x-onyxia": {
          "hidden": false,
          "overwriteDefaultWith": "region.nodeSelector",
          "overwriteSchemaWith": "nodeSelector.json"
      }
    }

```
{% endcode %}

The `overwriteDefaultWith` attribute was the old method for overriding, instructing Onyxia to use the "defaultConfiguration" from the Region. This method is no longer supported in v9, though it can still be used for catalog compatibility with v8.

In v9, `overwriteDefaultWith` has been replaced by `overwriteSchemaWith`, which offers more flexibility due to the capabilities of JSON Schemas. Default schemas are bundled with Onyxia-API and will be used if no override is provided. You can find these default schemas here: [Onyxia-API Schemas](https://github.com/InseeFrLab/onyxia-api/tree/main/onyxia-api/src/main/resources/schemas).

To override a schema, use the new `schemas` key from the v9 Helm chart and provide the list of schemas you want to override.\
For more details, refer to the documentation: [Onyxia v9 Catalog](https://docs.onyxia.sh/v/v9/admin-doc/catalog-of-services#x-onyxia-overwriteschemawith).



{% hint style="warning" %}
&#x20;Onyxia v9 will fail to start with error message :&#x20;

`FATAL : Setting defaultConfiguration in region is no longer supported and has been replaced by JSONSchema support. See migration guide at https://docs.onyxia.sh/admin-doc/migration-guides/v8-greater-than-v9`

&#x20;if you don't remove the `defaultConfiguration` from the region configuration.
{% endhint %}
