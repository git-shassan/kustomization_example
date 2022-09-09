# kustomization_example

Start with basic files for deployment (replica=1) and service ... Do not specify any label here, as that will be made a variable (instead of hard coded):

For example: 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ktest-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      # leaving blank
  template:
    metadata:
      labels:
      # leaving blank
    spec:
      containers:
      - image: nginx
        name: nginx
```

## Add label through kustomization:
In customization file, add the following lines. 
```
commonLabels:
  app: bingo
```

These will add lables at all the appropriate places. The "# leaving blank" comment in the files is not a place holder...it doesn't need to be even there. Kustomization knows where to insert labels and it will do that automatically. 
After we add the above lines, try rendering the files as shown here: 

```
oc kustomize files/

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bingo
  name: ktest-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```

Note the labels inserted now 

## Adding namespace.. 

If the base files are treated as templates, then each time they are applied it may be a different namespace. So namespace is not hard coded either. Its not even mentioned in the files. Instead, to apply this config to a different namespace on each run, the folliwing line can be added to kustomization.yaml:

```
namespace: my-namespace
```

Now if the files are rendered, a namespace is inserted in right place: 

```
oc kustomize files/

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bingo
  name: ktest-deployment
  namespace: my-namespace    <<<<
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```

## Unique runs within namespace

If the same files have to be run as multiple instances within a namespace, then the "name" will need to be different each time...
To achive this, use one or both of the following options in kustomization file: 

```
namePrefix: dev-
nameSuffix: "-001"
```

As a result, the rendered files will now have a prefix and suffix added to their names. For each subsequent run, either (or both) of these can be changed to create unique names for the deployments 

For example: 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bingo
  name: dev-ktest-deployment-001
  namespace: my-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```

## Changing other parameters:

All of the previous examples customize (or is it kustomize) the parameters related to metadata. There are also methods to modify annotations. However, none of these examples show changes to the specifications of a manifest. Once again, if the base files are meant to be used as templates, then subsequent runs may require changes to the specificications as well. This is achieved by adding patches. These patches can be added/removed/alterered on each run as desired. 

For example, in the current base file for deployment, there is only 1 replica being created. A patch file can be created to modify that, and then the patch definition can be added to kustomization file whenever desired. This is done as shown here: 

First, define the patch: 
```
cat files/replicas.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ktest-deployment
spec:
  replicas: 3
```
Now add the patch to kustomization.yaml:

```
patchesStrategicMerge:
 - replicas.yaml
```

Now try rendering the files, and notice that patch is in effect right away, and number of replicas shows up as 3:

```
oc kustomize files/

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bingo
  name: dev-ktest-deployment-001
  namespace: my-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```

