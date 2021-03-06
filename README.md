# Cloud Foundry Services example

This is an example of a service, a service gateway and an app in Cloud Foundry v2 and meant to illustrate how to utilize a service.
There are two blog posts describing the work in this repository.

[Introduction to Service and Service Gateway](http://pivotallabs.com/creating-a-service-gateway-in-cloud-foundry/)

[Describing how to set up the service so that it is usable in Cloud Foundry and binding it to an app](http://pivotallabs.com/creating-a-service-gateway-in-cloud-foundry-part-2)

The are three distinct parts in this repo:
1) The service (random_string_service)
2) The gateway (custom_gateway)
3) The app (sample_app)

The CustomService is a simple service that responds to a request for a random string, with the character count of the string as param.
The CustomGateway is needed to make the service available through Cloud Foundry.
The app is an example app to illustrate how all three are use in concert.

-- The following description expects a working Cloud Foundry deploy, where you have cf-admin rights --

## How to deploy the sample app with the RandomStringService
### Deploy the service as an app
```
$ cd [path_to]random_string_service
$ bundle
$cf push
$
$Name> random-string-service
$Instances> 1
$
$1: 128M
$2: 256M
$3: 512M
$4: 1G
$Memory Limit> 256M
$
$1: random-string-service
$2: none
$Subdomain> random-string-service
$
$1: georg.cf-app.com
$2: none
$Domain> georg.cf-app.com
$
$cf apps
$Getting applications in bootstrap-space... OK
$
$name                    status    usage      url
$random-string-service   running   1 x 256M   random-string-service.georg.cf-app.com
```

Get the url for the app for the config of the gateway to point to it.

### Deploy Custom Gateway

Before trying to push the gateway, make sure it is configured properly for your CF configuration
edit config/gateway_config.yml to point at correct URLs, and update its credentials accordingly.

Then create the service auth token and push

```
$ cd [path_to]custom_gateway
$ bundle
$ cf create-service-auth-token random-string-service pivotal --token 'random string service token'
$ cf push
```

wait until the custom-gateway is running  (check with `cf apps`)

### Deploy SampleApp

- make sure cf apps doesn't show custom-app
- create the custom service that the app will bind to
- this assumes the app manifest specifies that it wants to bind to that service

```
$ cf create-service random-string-service
> random-string-service-sample
$cf push

Binding sample-app.georg.cf-app.com to sample-app... OK

Create services for application?> y

1: random-string-service 1.0, via pivotal
2: rds-mysql n/a, via aws
3: user-provided , via
What kind?> 2

Name?> random-string-service-sample

1: 10req: Shared service, 10 requests
Which plan?> 1

Creating service random-string-service-sample... OK
Binding random-string-service-sample to sample-app... OK
Create another service?> n

Bind other services to application?> n
```
