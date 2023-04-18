---
title: "Passing Metadata Fields to Arguments in Kubernetes Yaml Files"
date: 2023-04-18T16:46:51+02:00
Description: ""
Tags: []
Categories: []
DisableComments: false
---

Passing metadata fields like `metadata.name` or `metadata.namespace` to the arguments field in Kubernetes YAML files is a powerful way to dynamically configure your containers and pods. This can be especially useful when you need to pass information about the pod or the environment to your application code.

The `fieldRef` syntax allows you to reference a specific field in the Kubernetes API object using the `fieldPath` attribute. This attribute takes a string that describes the path to the field in the object, using dot notation to separate nested fields.

Here's an example of how to pass the `metadata.name` field to the `arguments` field in a Kubernetes YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      command: ["/bin/sh"]
      args: ["-c", "echo My name is $(MY_POD_NAME)"]
      env:
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
```

In this example, I've set the value of the `MY_POD_NAME` environment variable to the value of the `metadata.name` field using the `fieldRef` syntax.

The `env` field in the container configuration specifies that the value of the `MY_POD_NAME` environment variable should be set using the `fieldRef` syntax. The `fieldPath` attribute specifies that the value of the environment variable should be set to the value of the `metadata.name` field.

Note that the `metadata.name` field is only available when the pod is running. If you need to set the value of the `MY_POD_NAME` environment variable before the pod starts up, you will need to use a different approach, such as using a Kubernetes init container to set the value of the environment variable.

Using the `fieldRef` syntax to pass metadata fields to the `arguments` field can be a powerful way to dynamically configure your containers and pods. With this approach, you can easily pass information about the pod or the environment to your application code, without having to hardcode the values in your YAML files.

Below, you can find another example:

```yaml
apiVersion: k6.io/v1alpha1
kind: K6
metadata:
  name: k6-ot-load-tests
  namespace: ot-load-tests
spec:
  parallelism: 3
  script:
    configMap:
      name: "ot-load-test-scripts"
      file: "scenario_get-watchlist.js"
  arguments:
    - -e
    - ENVIRONMENT=DEV
    - -e
    - PARALLELISM=3
    - -e
    - POD_NAME=$(POD_NAME)
  runner:
    env:
      - name: POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
```