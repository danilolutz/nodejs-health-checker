# nodejs-health-checker

[![License Status](https://img.shields.io/github/license/gritzkoo/nodejs-health-checker)](https://img.shields.io/github/license/gritzkoo/nodejs-health-checker)
[![Issues Status](https://img.shields.io/github/issues/gritzkoo/nodejs-health-checker)](https://img.shields.io/github/issues/gritzkoo/nodejs-health-checker)
[![Tag Status](https://img.shields.io/github/v/tag/gritzkoo/nodejs-health-checker)](https://img.shields.io/github/v/tag/gritzkoo/nodejs-health-checker)
[![Languages Status](https://img.shields.io/github/languages/count/gritzkoo/nodejs-health-checker)](https://img.shields.io/github/languages/count/gritzkoo/nodejs-health-checker)
[![Repo Size Status](https://img.shields.io/github/repo-size/gritzkoo/nodejs-health-checker)](https://img.shields.io/github/repo-size/gritzkoo/nodejs-health-checker)
[![GIT Downloads Status](https://img.shields.io/github/downloads/gritzkoo/nodejs-health-checker/total)](https://img.shields.io/github/downloads/gritzkoo/nodejs-health-checker/total)
[![NPM Downloads Status](https://img.shields.io/npm/dy/nodejs-health-checker)](https://img.shields.io/npm/dy/nodejs-health-checker)

This is a Node package that allows you to track the health of your application, providing two ways of checking:

*__Simple__*: will respond to a JSON as below and that allows you to check if your application is online and responding without checking any kind of integration.

```json
{
  "status": "fully functional"
}
```

*__Detailed__*: will respond a JSON as below and that allows you to check if your application is up and running and check if all of your integrations informed in the configuration list is up and running.

```json
{
    "name": "My node application",
    "version": "my version",
    "status": true,
    "date": "2020-09-18T15:29:41.616Z",
    "duration": 0.523,
    "integrations": [
        {
            "name": "redis integration",
            "kind": "Redis DB integration",
            "status": true,
            "response_time": 0.044,
            "url": "redis:6379"
        },
        {
            "name": "My memcache integration",
            "kind": "Memcached integraton",
            "status": true,
            "response_time": 0.038,
            "url": "memcache:11211"
        },
        {
            "name": "my web api integration",
            "kind": "Web integrated API",
            "status": true,
            "response_time": 0.511,
            "url": "https://github.com/status"
        }
    ]
}
```

## How to install

```sh
npm i nodejs-health-checker
```

## Available integrations

- [x] Redis
- [x] Memcached
- [x] Web integration (https)

## How to use

Example using Nodejs + Express

```javascript
import express from "express";
import {
  HealthcheckerDetailedCheck,
  HealthcheckerSimpleCheck
} from "./healthchecker/healthchecker";
import { HealthTypes } from "./interfaces/types";

const server = express();

server.get("/health-check/liveness", (_, res) => {
  res.send(HealthcheckerSimpleCheck());
});

server.get("/health-check/readiness", async (_, res) => {
  res.send(
    await HealthcheckerDetailedCheck({
      name: "My node application",
      version: "my version",
      // here you will inform all of your external dependencies
      // that your application must be checked to keep healthy
      // available integration types: [
      //   HealthTypes.Redis,
      //   HealthTypes.Memcached,
      //   HealthTypes.Web
      // ]
      integrations: [
        {
          type: HealthTypes.Redis,
          name: "redis integration",
          host: "redis",
        },
        {
          type: HealthTypes.Memcached,
          name: "My memcache integration",
          host: "memcache:11211",
        },
        {
          type: HealthTypes.Web,
          name: "my web api integration",
          host: "https://github.com/status",
          headers: [{ key: "Accept", value: "application/json" }],
        },
      ],
    })
  );
});

export default server;
```

And then, you could call this endpoints manually to see your application health, but, if you are using modern kubernetes deployment, you can config your chart to check your application with the setup below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: 'node' #your application image
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /health-check/liveness
        port: 80
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
  - name: readiness
    image: 'node' #your application image
    args:
    - /server
    readinessProbe:
      httpGet:
        path: /health-check/readiness
        port: 80
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
