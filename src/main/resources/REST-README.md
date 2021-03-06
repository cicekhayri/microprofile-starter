```
  __  __ _                _____            __ _ _      
 |  \/  (_)              |  __ \          / _(_) |     
 | \  / |_  ___ _ __ ___ | |__) | __ ___ | |_ _| | ___ 
 | |\/| | |/ __| '__/ _ \|  ___/ '__/ _ \|  _| | |/ _ \
 | |  | | | (__| | | (_) | |   | | | (_) | | | | |  __/
 |_|__|_|_|\___|_|  \___/|_|   |_|  \___/|_| |_|_|\___|
  / ____| |           | |                              
 | (___ | |_ __ _ _ __| |_ ___ _ __                    
  \___ \| __/ _` | '__| __/ _ \ '__|                   
  ____) | || (_| | |  | ||  __/ |                      
 |_____/ \__\__,_|_|   \__\___|_|                      
```

# Project generator REST API

This document summarizes examples of MicroProfile Starter web application REST API
[microprofile/starter/1](https://app.swaggerhub.com/apis/microprofile/starter/1)

Each MicroProfile version (mpVersion) offers a suite of specifications (specs) and is implemented
by a set of application servers (supportedServers).

In order to get a project zip generated one has to select at least the desired server (supportedServer).
The latest mpVersion with all specs it offers is used by default then.

# Get available MicroProfile versions

```
$ curl https://start.microprofile.io/api/1/mpVersion

["MP22","MP21","MP20","MP14","MP13","MP12"]
```

# Get available supportedServers and specs

Note the order of supportedServers values is pseudorandom each call.
Note the JSON formatting was altered for this document.

```
$ curl https://start.microprofile.io/api/1/mpVersion/MP14
{
  "supportedServers": [
    "PAYARA_MICRO",
    "LIBERTY",
    "TOMEE",
    "KUMULUZEE"
  ],
  "specs": [
    "CONFIG",
    "FAULT_TOLERANCE",
    "JWT_AUTH",
    "METRICS",
    "HEALTH_CHECKS",
    "OPEN_API",
    "OPEN_TRACING",
    "REST_CLIENT"
  ]
}
```

# Getting project

Curl: ```-O -J``` makes curl to save the zip file in the current directory. ```-L``` makes curl to follow redirects.

## Minimal example

Note that latest available MP version for the supportedServer and All available Specs were selected by default.

```
$ curl -O -J 'https://start.microprofile.io/api/1/project?supportedServer=THORNTAIL_V2'

curl: Saved to filename 'demo.zip'

```

## Examples

The only mandatory attribute is ```supportedServer```. One can omit ```-v``` curl flag.
Note the optional usage of [Etag](https://tools.ietf.org/html/rfc7232#section-2.3) and [If-None-Match](https://tools.ietf.org/html/rfc7232#section-3.2).

```
$ curl -v -O -J -L 'https://start.microprofile.io/api/1/project?supportedServer=PAYARA_MICRO&artifactId=XXXXX&mpVersion=MP12&selectedSpecs=FAULT_TOLERANCE&selectedSpecs=JWT_AUTH'

< ETag: "dd085c81"

curl: Saved to filename 'XXXXX.zip'
```

Note we can use ETag dd085c81 from the previous response:

```
curl -v -O -J -L -H 'If-None-Match: "dd085c81"' 'https://start.microprofile.io/api/1/project?supportedServer=PAYARA_MICRO&artifactId=XXXXX&mpVersion=MP12&selectedSpecs=FAULT_TOLERANCE&selectedSpecs=JWT_AUTH'

< HTTP/1.1 304 Not Modified
```

## Example with all attributes

```
$ curl -O -J -L 'https://start.microprofile.io/api/1/project?supportedServer=LIBERTY&groupId=com.example&artifactId=myapp&mpVersion=MP22&javaSEVersion=SE8&selectedSpecs=CONFIG&selectedSpecs=FAULT_TOLERANCE&selectedSpecs=JWT_AUTH&selectedSpecs=METRICS&selectedSpecs=HEALTH_CHECKS&selectedSpecs=OPEN_API&selectedSpecs=OPEN_TRACING&selectedSpecs=REST_CLIENT'

curl: Saved to filename 'myapp.zip'
```

## Use JSON for get the project

By POSTing a JSON string one can get the same result as with using query parameters above.

```
$ curl -O -J -L -H "Content-Type: application/json" -d '{"mpVersion":"MP14","supportedServer":"TOMEE","selectedSpecs":["REST_CLIENT","CONFIG"]}' 'https://start.microprofile.io/api/1/project'

curl: Saved to filename 'demo.zip'
```

This is how one can have the JSON stored in a file:

```
$ curl -O -J -L -H "Content-Type: application/json" -d @my.json 'https://start.microprofile.io/api/1/project'

curl: Saved to filename 'demo.zip'
```

## JSON All attributes used

```
$ cat all.json
{
  "groupId": "com.example",
  "artifactId": "myapp",
  "mpVersion": "MP22",
  "javaSEVersion": "SE8",
  "supportedServer": "THORNTAIL_V2",
  "selectedSpecs": [
    "CONFIG",
    "FAULT_TOLERANCE",
    "JWT_AUTH",
    "METRICS",
    "HEALTH_CHECKS",
    "OPEN_API",
    "OPEN_TRACING",
    "REST_CLIENT"
  ]
}

$ curl -O -J -L -H "Content-Type: application/json" -d @all.json 'https://start.microprofile.io/api/1/project'

curl: Saved to filename 'myapp.zip'
```

## JSON Minimal example

```
$ curl -O -J -L -H "Content-Type: application/json" -d '{"supportedServer":"KUMULUZEE"}' 'https://start.microprofile.io/api/1/project'

curl: Saved to filename 'demo.zip'
```

# Errors

Note that if you select an invalid combination of selectedSpecs and supportedServer and mpVersion you get an error, e.g.:

```
$ curl -v -O -J -L 'https://start.microprofile.io/api/1/project?supportedServer=HELIDON&selectedSpecs=REST_CLIENT'

curl: Saved to filename 'error.json'

$ cat error.json 
{"error":"One or more selectedSpecs is not available for given mpVersion","code":"ERROR003"}
```

What happened: We did not specify mpVersion, so the latest for the given server at the time was selected by default, it means MP12 at the time of writing of this example. MP12 does not offer spec REST_CLIENT as we can see:

```
$ curl https://start.microprofile.io/api/1/mpVersion/MP12
{
  "supportedServers": [
    "PAYARA_MICRO",
    "THORNTAIL_V2",
    "KUMULUZEE",
    "TOMEE",
    "HELIDON",
    "WILDFLY_SWARM",
    "LIBERTY"
  ],
  "specs": [
    "CONFIG",
    "FAULT_TOLERANCE",
    "JWT_AUTH",
    "METRICS",
    "HEALTH_CHECKS"
  ]
}
```

## Windows users

### curl.exe examples

Curl is widely available on Windows too, e.g. [https://curl.haxx.se/windows/](https://curl.haxx.se/windows/)

```
C:\curl\bin>curl -O -J https://start.microprofile.io/api/1/project?supportedServer=THORNTAIL_V2

curl: Saved to filename 'demo.zip'
```

```
C:\curl\bin>curl -O -J "https://start.microprofile.io/api/1/project?supportedServer=TOMEE&selectedSpecs=JWT_AUTH&selectedSpecs=CONFIG"

curl: Saved to filename 'demo.zip'
```

### Powershell examples

#### Minimal

```
PS C:\> Invoke-WebRequest -OutFile project.zip -Uri https://start.microprofile.io/api/1/project?supportedServer=LIBERTY
```

#### Using JSON

```
PS C:\> type .\all.json
{
  "groupId": "com.example",
  "artifactId": "myapp",
  "mpVersion": "MP22",
  "javaSEVersion": "SE8",
  "supportedServer": "THORNTAIL_V2",
  "selectedSpecs": [
    "CONFIG",
    "FAULT_TOLERANCE",
    "JWT_AUTH",
    "METRICS",
    "HEALTH_CHECKS",
    "OPEN_API",
    "OPEN_TRACING",
    "REST_CLIENT"
  ]
}
PS C:\> $headers = @{
>>   "Content-Type" = "application/json"
>> }
PS C:\> Invoke-WebRequest -InFile ./all.json -OutFile project.zip -Method Post -Uri "https://start.microprofile.io/api/1/project" -Headers $headers
```

# Integration with IDEs

If it is preferred to acquire all valid options in a one request the ```supportMatrix``` call is recommended.
The integration code [SHOULD](https://tools.ietf.org/html/rfc2119) use [Etag](https://tools.ietf.org/html/rfc7232#section-2.3) response header
and [If-None-Match](https://tools.ietf.org/html/rfc7232#section-3.2) request header so as to avoid unnecessary deserialization of identical responses.

## MicroProfile versions as keys

```
$ curl -i https://start.microprofile.io/api/1/supportMatrix
...
ETag: "44730639"
...
{
  "configs": {
    "MP21": {
      "supportedServers": [
        "THORNTAIL_V2",
        "PAYARA_MICRO",
        "KUMULUZEE",
        "LIBERTY"
      ],
      "specs": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ]
    },
    "MP22": {
      "supportedServers": [
        "PAYARA_MICRO",
        "LIBERTY",
        "KUMULUZEE",
        "THORNTAIL_V2"
      ],
      "specs": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ]
    },
  ...more MPxx removed for brevity...
  },
  "descriptions": {
    "CONFIG": "Configuration - externalize and manage your configuration parameters outside your microservices",
    "OPEN_API": "Open API - Generate OpenAPI-compliant API documentation for your microservices",
    "HEALTH_CHECKS": "Health Checks - Verify the health of your microservices with custom verifications",
    "REST_CLIENT": "Rest Client - Invoke RESTful services in a type-safe manner",
    "FAULT_TOLERANCE": "Fault Tolerance - all about bulkheads, timeouts, circuit breakers, retries, etc. for your microservices",
    "JWT_AUTH": "JWT Propagation - propagate security across your microservices",
    "OPEN_TRACING": "Open Tracing - trace the flow of requests as they traverse your microservices",
    "METRICS": "Metrics - Gather and create operational and business measurements for your microservices"
  }
}
```

Using value from ETag in If-None-Match:

```
$ curl '-HIf-None-Match: "44730639"' -i https://start.microprofile.io/api/1/supportMatrix
...
HTTP/1.1 304 Not Modified
```

## Server implementations as keys

```
$ curl -i https://start.microprofile.io/api/1/supportMatrix/servers
...
ETag: "7b99230f"
...
{
  "configs": {
    "PAYARA_MICRO": {
      "MP21": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP22": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP12": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS"
      ],
      "MP13": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP14": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP20": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ]
    },
    "THORNTAIL_V2": {
      "MP21": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP22": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ],
      "MP12": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS"
      ],
      "MP13": [
        "CONFIG",
        "FAULT_TOLERANCE",
        "JWT_AUTH",
        "METRICS",
        "HEALTH_CHECKS",
        "OPEN_API",
        "OPEN_TRACING",
        "REST_CLIENT"
      ]
    },
...more servers removed for brevity...
  },
  "descriptions": {
    "CONFIG": "Configuration - externalize and manage your configuration parameters outside your microservices",
    "OPEN_API": "Open API - Generate OpenAPI-compliant API documentation for your microservices",
    "HEALTH_CHECKS": "Health Checks - ensure your microservices is up and running by enabling health checks",
    "REST_CLIENT": "Rest Client - Invoke RESTful services in a type-safe manner",
    "FAULT_TOLERANCE": "Fault Tolerance - all about bulkheads, timeouts, circuit breakers, retries, etc. for your microservices",
    "JWT_AUTH": "JWT Propagation - manage and propagate security across your microservices",
    "OPEN_TRACING": "Open Tracing - trace the flow of requests as they traverse your microservices",
    "METRICS": "Metrics - Gather and create operational and business measurements for your microservices"
  }
}
```

Using value from ETag in If-None-Match:

```
$ curl '-HIf-None-Match: "7b99230f"' -i https://start.microprofile.io/api/1/supportMatrix/servers
...
HTTP/1.1 304 Not Modified
```
