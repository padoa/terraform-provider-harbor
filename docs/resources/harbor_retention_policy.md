# Resource: harbor_retention_policy

Harbor official documentation feature reference: https://goharbor.io/docs/2.0.0/working-with-projects/working-with-images/create-tag-retention-rules/

## Example Usage

```hcl
resource "harbor_project" "main" {
    name = "main"
}

resource "harbor_retention_policy" "cleanup" {
    scope {
        ref = harbor_project.main.id
    }

    rule {
        template = "always"
        tag_selectors {
            decoration = "matches"
            pattern    = "master"
            extras     = jsonencode({
                untagged: false
            })
        }
        scope_selectors {
            repository {
                kind       = "doublestar"
                decoration = "repoMatches"
                pattern    = "**"
            }
        }
    }

    rule {
        disabled = true
        template = "latestPulledN"
        params = {
            "latestPulledN"      = 15
            "nDaysSinceLastPush" = 7
        }
        tag_selectors {
            kind       = "doublestar"
            decoration = "excludes"
            pattern    = "master"
            extras     = jsonencode({
                untagged: false
            })
        }
        scope_selectors {
            repository {
                kind       = "doublestar"
                decoration = "repoExcludes"
                pattern    = "nginx-*"
            }
        }
    }

    trigger {
        settings {
            cron = "0 0 0 * * *"
        }
    }
}
```

## Argument Reference

The following arguments are supported:

* `algorithm` - (Optional) Algorithm used to aggregate rules. Defaults to `"or"`.

* `scope` - (Required) Describes the (project) scope of this retention policy.

* `rule` - (Required) List of retention rules that compose the policy.

* `trigger` - (Required) Defines a schedule with a cron rule.

## Nested Blocks

### scope

#### Arguments

* `level` - (Optional) Defines the level the scope applies to. Defaults to `"project"`.

* `ref` - (Required) Reference id (usually project id) the retention policy refers to.

### rule

#### Arguments

* `disabled` - (Optional) Specifies if the rule should be taken in account or not (ie disabled). Defaults to `false`.

* `action` - (Optional) Specifies the action to do if set of rules is matched. Defaults to `"retain"`.

* `template` - (Required) kind of rules among the following:

  * `latestPushedK`: (retain) the most recently pushed # artifacts

  * `latestPulledN`: (retain) the most recently pulled # artifacts

  * `nDaysSinceLastPush`: (retain) the artifacts pushed with the last # days

  * `nDaysSinceLastPull`: (retain) the artifacts pulled with the last # days

  * `always`: (retain) always

* `params` - (Optional) Map of params for the kind of rule (number of days or artifacts associated to the rule template). Defaults to empty map.

Note that multiple key-values can be specified instead of only the one matching the template. The other ones would be disabled but recorded as default choice to the associated template within the UI combo-box.

* `tag_selectors` - (Required) Describes the image tags on which to apply the rule.

* `scope_selectors` - (Required) Describes the image name(s) on which to apply the rule.

#### Attributes

* `id` - The id of the rule (currently not used and set to `0` whatever the number of rules).

* `priority` - A priority tag to know in which order the set of rules should b performed (currently not used and set to `0`).

### `tag_selectors`

#### Arguments

* `kind` - (Optional) The kind of tag selector. Defaults to `doublestar`.

* `extras` - (Optional) Describes extra info for tag selector as a json encoded string, supports mainly the following key value: `untagged: (false|true)` to match or not the untagged artifacts in the rule. Defaults to `""`.

* `decoration` - (Required) Specifies if the pattern should be matched (`"matches"`) or excluded (`"excludes"`).

* `pattern` - (Required) Describes the pattern that should match (or exclude) image tags

### `scope_selectors`

#### Arguments

* `kind` - (Optional) The kind of scope selector. Defaults to `doublestar`.

* `extras` - (Optional) Describes extra info for scope selector as a json encoded string, does not support any known key value as now. Defaults to `""`.

* `decoration` - (Required) Specifies if the pattern should be matched (`"repoMatches"`) or excluded (`"repoExcludes"`).

* `pattern` - (Required) Describes the pattern that should match (or exclude) image names

### `trigger`

#### Arguments

* `kind` - (Optional) The kind of trigger. Defaults to `Schedule`.

* `settings` - (Required) The trigger settings.

#### Attributes

* `references` - Reference to an internal job responsible for applying the trigger.

### `settings`

#### Arguments

* `cron` - (Optional) 6-fields cron pattern for triggering the retention policy execution by cron schedule. Defaults to `""` (which refers to a "None" Schedule).

### `references`

#### Attributes

* `job_id` - Reference to the internal job id associated to the trigger.

## Attributes Reference

In addition to all arguments, the following attribute is exported:

* `id` - The id of the retention policy.

## Import

Harbor Retention policies can be imported using the `harbor_retention_policy`, e.g.

```sh
terraform import harbor_retention_policy.cleanup 2
```
