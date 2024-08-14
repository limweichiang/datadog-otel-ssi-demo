# Objective

The objective of this is to demonstrate and compare Datadog APM distributed trace collection for an unmodified Java application under the following scenarios:
- Datadog Agent + Datadog APM Single Step Instrumentation
- Datadog Agent with OTLP ingest + OpenTelemetry Automatic Instrumentation

The following illustration demonstrates both paths that will be taken

![]()

# Prerequisites

- A Kubernetes Cluster - This was tested against an AWS EKS Cluster of 2 nodes
- `helm` and `kubectl` installed on a workspace with access to the Kubernetes Cluster - MacOS was used in this scenario
- Browser on the workspace - Chrome was used on MacOS in this scenario

# Deployment Steps 

## General

Clone this repository into your workspace
```
git clone
```

## Scenario 1: Datadog Agent + Datadog APM Single Step Instrumentation

Install the Datadog Operator, then deploy the Datadog Agent with Single Step APM Instrumentation:
```
$ helm install datadog-operator datadog/datadog-operator
$ kubectl apply -f datadog-demo/datadog-operator-datadog-config.yaml 
```

Deploy a Java application (Liferay) with Single Step APM Instrumentation:
```
$ kubectl apply -f datadog-demo/liferay-datadog-java-autoinstrumentation-deployment.yaml
```

Set up port-forwarding to allow access to test the application:
```
$ kubectl port-forward svc/liferay-service 8080:80
```

Use your browser of choice to acess `http://localhost:8080`, then perform some simple actions with the application. This is simply to generate APM traces, which you should then see on the Datadog App.


## Scenario 2: Datadog Agent with OTLP ingest + OpenTelemetry Automatic Instrumentation

Install the Datadog Operator, then deploy the Datadog Agent with OTLP receivers configured.
```
$ helm install datadog-operator datadog/datadog-operator
$ kubectl apply -f otel-demo/datadog-operator-otel-config.yaml
```

Install the OpenTelemetry Operator. Note that this does NOT install the Otel Collector:
```
$ kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
```

Deploy OpenTelemetry's Java instrumentation resource:
```
$ kubectl apply -f otel-demo/otel-java-autoinstrumentation-instrumentation-crd.yaml
```
Note that Automatic Instrumentation is applied on a per-language basis for OpenTelemetry. This definition only deploys the Java instrumentation resource. Definitions for other languages can be refered from the ![OpenTelemetry Injecting Auto-Instrumentation documentation](https://opentelemetry.io/docs/kubernetes/operator/automatic/#configure-automatic-instrumentation)

Deploy a Java application (Liferay) with OpenTelemetry Automatic Instrumentation for Java:
```
kubectl apply -f otel-demo/liferay-otel-java-autoinstrumentation-deployment.yaml
```

Set up port-forwarding to allow access to test the application:
```
$ kubectl port-forward svc/liferay-service 8080:80
```

Use your browser of choice to acess `http://localhost:8080`, then perform some simple actions with the application. This is simply to generate APM traces, which you should then see on the Datadog App.