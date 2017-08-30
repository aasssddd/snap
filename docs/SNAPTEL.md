<!--
http://www.apache.org/licenses/LICENSE-2.0.txt


Copyright 2015 Intel Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# snaptel
A powerful telemetry framework

## Usage
The `snap-telemetry` package installs `snaptel` in `/usr/local/sbin/snaptel`, and `snapteld` in `/usr/local/sbin/snapteld`. Either ensure `/usr/local/bin:/usr/local/sbin` is in your path, or use fully qualified filepath to the `snaptel`/`snapteld` binary:

```
$ snaptel [global options] command [command options] [arguments...]
```

### Global Options
```
--url, -u 'http://localhost:8181'    Sets the URL to use [$SNAP_URL]
--insecure                           Ignore certificate errors when Snap's API is running HTTPS [$SNAP_INSECURE]
--api-version, -a 'v1'               The Snap API version [$SNAP_API_VERSION]
--password, -p                       Require password for REST API authentication [$SNAP_REST_PASSWORD]
--config, -c                         Path to a config file [$SNAPTEL_CONFIG_PATH]
--help, -h                           show help
--version, -v                        print the version
```

### Commands
```
metric
plugin
task
help, h      Shows a list of commands or help for one command
```

### Command Options

#### task
```
$ snaptel task command [command options] [arguments...]
```
```
create      There are two ways to create a task.
              1) Use a task manifest with [--task-manifest, t]
              2) Provide a workflow manifest and schedule details [--workflow-manifest, -w]

              --task-manifest value, -t value      File path for task manifest to use for task creation.
              --workflow-manifest value, -w value  File path for workflow manifest to use for task creation
              --interval value, -i value           Interval for the task schedule [ex (simple schedule): 250ms, 1s, 30m (cron schedule): "0 * * * * *"]
	          --count value                        The count of runs for the task schedule [defaults to 0 what means no limit, e.g. set to 1 determines a single run task]
              --start-date value                   Start date for the task schedule [defaults to today]
              --start-time value                   Start time for the task schedule [defaults to now]
              --stop-date value                    Stop date for the task schedule [defaults to today]
              --stop-time value                    Start time for the task schedule [defaults to now]
              --name value, -n value               Optional requirement for giving task names
              --duration value, -d value           The amount of time to run the task [appends to start or creates a start time before a stop]
              --no-start                           Do not start task on creation [normally started on creation]
              --deadline value                     The deadline for the task to be killed after started if the task runs too long (All tasks default to 5s)
              --max-failures value                 The number of consecutive failures before Snap disables the task

            * Note: Start and stop date/time are optional.
list        list
start       start <task_id>
stop        stop <task_id>
remove      remove <task_id>
export      export <task_id>
watch       watch <task_id>
enable      enable <task_id>
help, h     Shows a list of commands or help for one command
```

#### plugin
```
$ snaptel plugin command [command options] [arguments...]
```
```
load        load <plugin_path> [--plugin-cert=<plugin_cert_path> --plugin-key=<plugin_key_path> --plugin-ca-certs=<ca_cert_paths>]
unload      unload <plugin_type> <plugin_name> <plugin_version>
swap        swap <load_plugin_path> <unload_plugin_type>:<unload_plugin_name>:<unload_plugin_version> or swap <load_plugin_path> -t <unload_plugin_type> -n <unload_plugin_name> -v <unload_plugin_version> [--plugin-cert=<plugin_cert_path> --plugin-key=<plugin_key_path> [--plugin-ca-certs=<ca_cert_paths>] ]
list        list
help, h     Shows a list of commands or help for one command
```

#### metric
```
$ snaptel metric command [command options] [arguments...]
```
```
list         list
get          get details on a single metric
help, h      Shows a list of commands or help for one command
```

Example Usage
-------------

### Load and unload plugins, create and start a task

In one terminal window, run snapteld (log level is set to 1 and signing is turned off for this example):
```
$ snapteld --log-level 1 --log-path '' --plugin-trust 0
```

prepare a task manifest file, for example, task.json with following content:
```json
{
    "version": 1,
    "schedule": {
        "type": "simple",
        "interval": "1s"
    },
    "max-failures": 10,
    "workflow": {
        "collect": {
            "metrics": {
                "/intel/mock/foo": {},
                "/intel/mock/bar": {},
                "/intel/mock/*/baz": {}
            },
            "config": {
                "/intel/mock": {
                    "name": "root",
                    "password": "secret"
                }
            },
            "process": [
                {
                    "plugin_name": "passthru",
                    "process": null,
                    "publish": [
                        {
                            "plugin_name": "mock-file",
                            "config": {
                                "file": "/tmp/snap_published_mock_file.log"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```

prepare a workflow manifest file, for example, workflow.json with the following content:
```json
{
    "collect": {
        "metrics": {
            "/intel/mock/foo": {}
        },
        "config": {
            "/intel/mock/foo": {
                "password": "testval"
            }
        },
        "process": [],
        "publish": [
            {
                "plugin_name": "mock-file",
                "config": {
                    "file": "/tmp/rest.test"
                }
            }
        ]
    }
}
```

and then:

1. load a collector plugin
2. load a processing plugin
3. load a publishing plugin
4. list the plugins
5. start a task with a task manifest
6. start a single run task with a task manifest
7. start a task with a workflow manifest
8. list the tasks
9. unload the plugins

```
$ snaptel plugin load /opt/snap/plugins/snap-plugin-collector-mock1
$ snaptel plugin load /opt/snap/plugins/snap-plugin-processor-passthru
$ snaptel plugin load /opt/snap/plugins/snap-plugin-publisher-mock-file
$ snaptel plugin list
$ snaptel task create -t mock-file.json
$ snaptel task create -t mock-file.json --count 1
$ snaptel task create -w workflow.json -i 1s -d 10s
$ snaptel task list
$ snaptel plugin unload collector mock <version>
$ snaptel plugin unload processor passthru <version>
$ snaptel plugin unload publisher publisher <version>
```

## More information
* [SECURE_PLUGIN_COMMUNICATION](SECURE_PLUGIN_COMMUNICATION.md)