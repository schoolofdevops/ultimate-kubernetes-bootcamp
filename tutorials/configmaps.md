# Configmaps
Configmap is one of the ways to provide configurations to your application.

Check our configmap for vote app

```
cat apps/voting/dev/vote-config.yaml


apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: dev
data:
  OPTION_A: EMACS
  OPTION_B: VI
```

In the above given configmap, we define two environment variables,
  1. OPTION_A=EMACS
  2. OPTION_B=VI

In order to use this configmap in the deployment, we need to reference it from the deployment file.
Check the deployment file for vote add for the following block.

```
envFrom:
  - configMapRef:
      name: vote
```

So when you create your deployment, these configurations will be made available to your application. In this example, the values defined in the configmap (EMACS and VI) will override the default values(CATS and DOGS) present in your source code.

![configmap](images/Configmap.png "ConfigMap")
