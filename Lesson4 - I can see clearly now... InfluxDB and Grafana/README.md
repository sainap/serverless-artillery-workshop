# Lesson 4: I can see clearly now… InfluxDB and Grafana
Goal: Add the InfluxDB plugin and view your test results using Grafana

This next step will involve adding dependencies to the package.json so that the testing Lambda can write results to InfluxDB.

### Step 1: Create a custom service for your tests

This greater level of customization allows the modification of the code which runs in AWS Lambda.

```sh
$ mkdir customTestLambda
$ cd customTestLambda
$ cp ../script.yml .
$ slsart configure
$ ls
handler.js	package.json    serverless.yml
```

(Note you may also see a script.yml if you've previously created that file.)

The purposes of these files follow:

|file|description|
|:----|:----------|
|package.json|Node.js dependencies for the Lambda.  Add Artillery plugins you want to use here.|
|serverless.yml|Serverless service definition. Change AWS-specific settings here|
|handler.js|Code to implement the load testing. **EDIT AT YOUR OWN RISK**|

From now on, slsart deploy and run will use these configuration files when run in this directory.  To use the originals, switch to a directory without a `serverless.yml`.

This entire directory is uploaded to AWS when the Lambda is deployed. This allows you to add plugins or payload files (*.csv).  Make sure that after any modifications to any of the files or the addition of new files, that you re-deploy with:

```sh
slsart deploy
```

Optionally, it should be possible to test that the current directory configuration is working with:

```sh
$ slsart deploy
$ slsart invoke -s script.yml
```

This should deploy and invoke the lambda using the local `./script.yml`.

### Step 2: Add influxdb Artillery plugin

To allow the Lambda code to write to InfluxDB, the correct NPM package dependency must be added. Run the following command (no -g global flag!) in the directory just created:

```sh
npm install --save artillery-plugin-influxdb
```

This modifies the package.json file to include the necessary dependency. The package.json file should now contain all these dependencies (version numbers may vary):

```JSON
{
  "dependencies": {
    "artillery-core": "^2.0.3-0",
    "artillery-plugin-influxdb": "^0.6.1",
    "csv-parse": "^1.1.7"
  }
}
```

### Step 3: Update Script to Log to Influx Server


Update the `config` portion of the test script to add the `plugin` from the example below:

```sh
config:
  plugins:
    influxdb:
      testName: "<TEST_CASE_NAME>"  # This name must be changed
      influx:
        host: "ec2-54-86-68-145.compute-1.amazonaws.com"
        username: "admin"
        password: "admin"
        database: "artillery_metrics"

```

Lambda now has dependencies added to the node_modules directory, so it's necessary to upload it again.
Then it can run again with the newly updated script:

```sh
$ slsart deploy
$ slsart invoke -s script.yml
```

If all went correctly, the load test data was added to the InfluxDB. Next step is to query the DB.

### Step 4: Query and Visualize the Results

The database containing the results should have already been created, and is referenced in the test script under the `influx` part. 
In this example, the database `artillery_metrics` is used.


Log into InfluxDB at: [http://ec2-54-86-68-145.compute-1.amazonaws.com:8083/](http://ec2-54-86-68-145.compute-1.amazonaws.com:8083/) and perform a quick query to check that database exists:

```
SHOW DATABASES
```

The `artillery_metrics` database should be listed. In the InfluxDB dashboard, select the `artillery_metrics` database from the drop-down list in the upper-right corner.
With that selection made, queries made will be against that database.
 
To see the measurements stored in this database, run this command:
  
```
SHOW MEASUREMENTS
```  

The `latency` measurement should be in the list. To show all of the latencies, select them all:

```
SELECT * FROM latency
```

To see only results from a specific test, run:
 
```
SELECT * FROM latency WHERE testName = '<TEST_CASE_NAME>' limit 10
```

Once the test results have been verified in InfluxDB, it's time to see the graphs in Grafana.

[http://ec2-54-86-68-145.compute-1.amazonaws.com:3000](http://ec2-54-86-68-145.compute-1.amazonaws.com:3000)

Log in using `admin/admin` for the username/password and open the `Load Test Results` dashboard.
Once on the dashboad, pick the test name from the drop-down list, make sure that the time-span includes the test results above.
 
There will be a visualization of the test results, including latencies load and errors.

![Load Test Dashboard](https://github.com/Nordstrom/serverless-artillery-workshop/blob/master/Images/grafana-dashboard.jpg)

#### Optional:

Set Grafana to update every 10 seconds and show metrics from the last minute or so, by clicking on the clock at the right of the nav bar at the top. Return to the console and run the tests again,
perhaps increasing the test duration to a minute long or more.

Switch back to the Grafana dashboard and watch the test results in real-time!

### Sample Grafana Dashboard

Here's the JSON definition of a Grafana Dashboard to start from:

```JSON
{
  "id": 12,
  "title": "Load Test Results Copy",
  "originalTitle": "Load Test Results",
  "tags": [],
  "style": "dark",
  "timezone": "browser",
  "editable": true,
  "hideControls": false,
  "sharedCrosshair": false,
  "rows": [
    {
      "collapse": false,
      "editable": true,
      "height": "250px",
      "panels": [
        {
          "aliasColors": {
            "client error": "#BF1B00",
            "errors": "#BF1B00",
            "latency": "#1F78C1",
            "load": "#508642",
            "responses": "#508642",
            "server errors": "#EF843C"
          },
          "bars": false,
          "datasource": "artillery-monitoring",
          "editable": true,
          "error": false,
          "fill": 1,
          "grid": {
            "threshold1": null,
            "threshold1Color": "rgba(216, 200, 27, 0.27)",
            "threshold2": null,
            "threshold2Color": "rgba(234, 112, 112, 0.22)"
          },
          "id": 3,
          "interval": "10s",
          "isNew": true,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 2,
          "links": [],
          "nullPointMode": "null",
          "percentage": false,
          "pointradius": 1,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [
            {
              "alias": "load",
              "yaxis": 1
            },
            {
              "alias": "errors",
              "yaxis": 2
            },
            {
              "alias": "client error",
              "yaxis": 1
            },
            {
              "alias": "server errors",
              "yaxis": 1
            },
            {
              "alias": "latency",
              "yaxis": 2
            }
          ],
          "span": 12,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "alias": "latency",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT mean(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "A",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "mean"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "responses",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT count(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "B",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "count"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "server errors",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "hide": false,
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT count(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND \"response\" <> '200' AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "C",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "count"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "response",
                  "operator": "=",
                  "value": "200"
                }
              ]
            },
            {
              "alias": "client error",
              "dsType": "influxdb",
              "groupBy": [],
              "measurement": "clientErrors",
              "policy": "sum",
              "query": "SELECT sum(\"value\") FROM \"clientErrors\" WHERE  \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "D",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  }
                ]
              ],
              "tags": []
            }
          ],
          "timeFrom": null,
          "timeShift": null,
          "title": "Responses and Errors",
          "tooltip": {
            "msResolution": true,
            "shared": true,
            "sort": 0,
            "value_type": "cumulative"
          },
          "type": "graph",
          "xaxis": {
            "show": true
          },
          "yaxes": [
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            },
            {
              "format": "ms",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            }
          ]
        }
      ],
      "title": "Row"
    },
    {
      "collapse": false,
      "editable": true,
      "height": "200px",
      "panels": [
        {
          "aliasColors": {
            "200": "#1F78C1"
          },
          "bars": false,
          "datasource": "artillery-monitoring",
          "editable": true,
          "error": false,
          "fill": 1,
          "grid": {
            "threshold1": null,
            "threshold1Color": "rgba(216, 200, 27, 0.27)",
            "threshold2": null,
            "threshold2Color": "rgba(234, 112, 112, 0.22)"
          },
          "id": 2,
          "interval": "10s",
          "isNew": true,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 1,
          "links": [],
          "nullPointMode": "null",
          "percentage": false,
          "pointradius": 5,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [
            {
              "alias": "All",
              "yaxis": 1
            },
            {
              "alias": "500",
              "yaxis": 1
            },
            {
              "alias": "200",
              "yaxis": 1
            }
          ],
          "span": 12,
          "stack": false,
          "steppedLine": true,
          "targets": [
            {
              "alias": "All",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT count(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": false,
              "refId": "A",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "count"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "500",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT count(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND \"response\" = '500' AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "B",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "count"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "response",
                  "operator": "=",
                  "value": "500"
                }
              ]
            },
            {
              "alias": "200",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT count(\"value\") FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND  \"response\" = '200' AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "C",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "count"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "response",
                  "operator": "=",
                  "value": "200"
                }
              ]
            }
          ],
          "timeFrom": null,
          "timeShift": null,
          "title": "Server Responses",
          "tooltip": {
            "msResolution": true,
            "shared": true,
            "sort": 0,
            "value_type": "cumulative"
          },
          "type": "graph",
          "xaxis": {
            "show": true
          },
          "yaxes": [
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            },
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": false
            }
          ]
        }
      ],
      "title": "New row"
    },
    {
      "collapse": false,
      "editable": true,
      "height": "500px",
      "panels": [
        {
          "aliasColors": {
            "P90": "#EAB839",
            "P95": "#EF843C",
            "max": "#E24D42",
            "mean": "#629E51",
            "min": "#3F6833"
          },
          "bars": false,
          "datasource": "artillery-monitoring",
          "editable": true,
          "error": false,
          "fill": 0,
          "grid": {
            "threshold1": null,
            "threshold1Color": "rgba(216, 200, 27, 0.27)",
            "threshold2": null,
            "threshold2Color": "rgba(234, 112, 112, 0.22)"
          },
          "id": 1,
          "interval": "1s",
          "isNew": true,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 0,
          "links": [],
          "nullPointMode": "null",
          "percentage": false,
          "pointradius": 5,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [
            {
              "alias": "max",
              "fillBelowTo": "P95",
              "lines": false
            },
            {
              "alias": "P95",
              "fillBelowTo": "P90",
              "lines": false
            },
            {
              "alias": "P90",
              "fillBelowTo": "mean",
              "lines": false
            },
            {
              "alias": "mean",
              "fillBelowTo": "min",
              "lines": false
            },
            {
              "alias": "min",
              "fillBelowTo": "min",
              "lines": false
            },
            {
              "alias": "mean",
              "lines": true,
              "linewidth": 1
            },
            {
              "alias": "min",
              "lines": true,
              "linewidth": 1
            },
            {
              "alias": "max",
              "lines": true,
              "linewidth": 1
            }
          ],
          "span": 12,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "alias": "max",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "refId": "A",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "max"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "P95",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT percentile(\"value\", 95) FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "D",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "max"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "P90",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "query": "SELECT percentile(\"value\", 90) FROM \"latency\" WHERE \"testName\" =~ /^$testName$/ AND $timeFilter GROUP BY time($interval) fill(null)",
              "rawQuery": true,
              "refId": "E",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "max"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "mean",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "refId": "C",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "mean"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            },
            {
              "alias": "min",
              "dsType": "influxdb",
              "groupBy": [
                {
                  "params": [
                    "$interval"
                  ],
                  "type": "time"
                },
                {
                  "params": [
                    "null"
                  ],
                  "type": "fill"
                }
              ],
              "measurement": "latency",
              "policy": "default",
              "refId": "B",
              "resultFormat": "time_series",
              "select": [
                [
                  {
                    "params": [
                      "value"
                    ],
                    "type": "field"
                  },
                  {
                    "params": [],
                    "type": "min"
                  }
                ]
              ],
              "tags": [
                {
                  "key": "testName",
                  "operator": "=~",
                  "value": "/^$testName$/"
                }
              ]
            }
          ],
          "timeFrom": null,
          "timeShift": null,
          "title": "Latency",
          "tooltip": {
            "msResolution": true,
            "shared": true,
            "sort": 0,
            "value_type": "cumulative"
          },
          "type": "graph",
          "xaxis": {
            "show": true
          },
          "yaxes": [
            {
              "format": "ms",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            },
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            }
          ]
        }
      ],
      "title": "New row"
    }
  ],
  "time": {
    "from": "2016-07-27T19:08:31.863Z",
    "to": "2016-07-27T19:15:44.210Z"
  },
  "timepicker": {
    "refresh_intervals": [
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "templating": {
    "list": [
      {
        "current": {
          "text": "Load-Test-01",
          "value": "Load-Test-01"
        },
        "datasource": "artillery-monitoring",
        "hide": 0,
        "includeAll": false,
        "label": "Test Name",
        "multi": false,
        "name": "testName",
        "options": [],
        "query": "show tag values from latency with key = testName",
        "refresh": 2,
        "type": "query"
      }
    ]
  },
  "annotations": {
    "list": []
  },
  "refresh": false,
  "schemaVersion": 12,
  "version": 25,
  "links": []
}
```