---
template: main.html
---

# `metrics/collect`
The `metrics/collect` task enables collection of [built-in metrics](../../metrics/builtin.md). It generates a stream of HTTP requests to one or more app/ML model versions, and collects latency and error metrics.

## Example

The following start action contains a `metrics/collect` task which is executed at the start of the experiment. The task sends a certain number of HTTP requests to each version specified in the task, and collects built-in latency/error metrics for them.

```yaml
start:
- task: metrics/collect
  with:
    versions:
	# Version names must be unique. 
	# If the version name matches the name of a version in the experiment's `versionInfo` section, 
	# then the version is considered *real*. 
	# If the version name does not match the name of a version in the experiment's `versionInfo` section, 
	# then the version is considered *pseudo*. 
	# Built-in metrics collected for real versions can be used within the experiment's `criteria` section. 
	# Pseudo versions are useful if the intent is only to generate load (`GET` and `POST` requests). 
	# Built-in metrics collected for pseudo versions cannot be used with the experiment's `criteria` section.
    - name: iter8-app
      # URL is where this version receives HTTP requests
      url: http://iter8-app.default.svc:8000
    - name: iter8-app-candidate
      url: http://iter8-app-candidate.default.svc:8000
```

## Inputs
| Field name | Field type | Description | Required |
| ----- | ---- | ----------- | -------- |
| time | string | Duration of the `metrics/collect` task run. Specified in the [Go duration string format](https://golang.org/pkg/time/#ParseDuration). Default value is `5s`. | No |
| payloadURL | string | URL of JSON-encoded data. If this field is specified, the metrics collector will send HTTP POST requests to versions, and the POST requests will contain this JSON data as payload. | No |
| versions | [][Version](#version) | A non-empty list of versions. | Yes |
| loadOnly | bool | If set to true, this task will send requests without collecting metrics. Default value is `false`. | No |

### Version
| Field name | Field type | Description | Required |
| ----- | ---- | ----------- | -------- |
| name | string | Name of the version. Version names must be unique. If the version name matches the name of a version in the experiment's `versionInfo` section, then the version is considered *real*. If the version name does not match the name of a version in the experiment's `versionInfo` section, then the version is considered *pseudo*. Built-in metrics collected for real versions can be used within the experiment's `criteria` section. Pseudo versions are useful if the intent is only to generate load (`GET` and `POST` requests). Built-in metrics collected for pseudo versions cannot be used with the experiment's `criteria` section. | Yes |
| qps | float | How many queries per second will be sent to this version. Default is 8.0. | No |
| headers | map[string]string | HTTP headers to be used in requests sent to this version. | No |
| url | string | HTTP URL of this version. | Yes |


## Result

This task will run for the specified duration (`time`), send requests to each version (`versions`) at the specified rate (`qps`), and will collect [built-in metrics]() for each version. Built-in metric values are stored in the metrics field of the experiment status in the same manner as custom metric values.

The task may result in an error, for instance, if one or more required fields are missing or if URLs are mis-specified. In this case, the experiment to which it belongs will fail.

## Start vs loop actions
If this task is embedded in start actions, it will run once at the beginning of the experiment.

If this task is embedded in loop actions, it will run in each loop of the experiment. The results from each run will be aggregated.

## Load generation without metrics collection
You can use this task to send HTTP GET and POST requests to app/ML model versions without collecting metrics by setting the [`loadOnly` input](#inputs) to `true`.