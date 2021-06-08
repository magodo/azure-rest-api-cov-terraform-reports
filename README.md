## Intro

This repo stores the coverage reports, which are intended to be consumed by the **azure-rest-api-cov-viewer**.

Especially, each of the reports represent:

- tf.json: The output of the **azure-rest-api-cov-terraform**, which represents the used API operations and its properties by Terraform, for a certain version of the AzureRM provider.
- api-cov.json: The output of `azure-rest-api-cov coverage` by consuming the `tf.json`, with coverage prediction on newer API versions. Currently, the output will be filtered by `jq` (`'to_entries | del(.[].value | .[].operations.GET)|from_entries'`) to remove the `GET` operations, for the sake of space saving.
- api.json: The output of `azure-rest-api-cov grant | azure-rest-api-cov report`, by consuming the `api-cov.json`.

## Tips

1. List the uncovered properties for a certain (`rp`, `api_path`, `operation`, `version`):

    ```
    $ jq --arg p $p --arg v $v --arg rp $rp --arg op $op '.[$rp] | select(.[$p] != null) | .[$p].operations | .[$op] | .[$v] |.swagger_rel_path, (.properties | to_entries[] | select(.value.state == "Uncovered") | .key) ' api-cov.json
    ```


    Example (Linux):

    ```
    $ rp=alertsmanagement
    $ p="/SUBSCRIPTIONS/{}/RESOURCEGROUPS/{}/PROVIDERS/MICROSOFT.ALERTSMANAGEMENT/SMARTDETECTORALERTRULES/{}"
    $ op=PUT
    $ v="2021-04-01"
    $ jq --arg p $p --arg v $v --arg rp $rp --arg op $op '.[$rp] | select(.[$p] != null) | .[$p].operations | .[$op] | .[$v] |.swagger_rel_path, (.properties | to_entries[] | select(.value.state == "Uncovered") | .key)' api-cov.json
    "alertsmanagement/resource-manager/Microsoft.AlertsManagement/stable/2021-04-01/SmartDetectorAlertRulesApi.json"
    "properties/detector/parameters"
    ```

    Example (Windows):

    ```
    > $rp="alertsmanagement"
    > $p="/SUBSCRIPTIONS/{}/RESOURCEGROUPS/{}/PROVIDERS/MICROSOFT.ALERTSMANAGEMENT/SMARTDETECTORALERTRULES/{}"
    > $op="PUT"
    > $v="2021-04-01"
    > .\jq-win64.exe --arg p $p --arg v $v --arg rp $rp --arg op $op '.[$rp] | select(.[$p] != null) | .[$p].operations | .[$op] | .[$v] |.swagger_rel_path, (.properties | to_entries[] | select(.value.state == \"Uncovered\") | .key)' api-cov.json
    ```
