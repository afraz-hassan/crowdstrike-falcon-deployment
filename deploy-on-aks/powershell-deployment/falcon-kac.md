# Falcon KAC (Kubernetes Admission Controller) Deployment
Welcome to the falcon kubernetes admission controller deployment. In this file we will be using powershell related commands.

## Step 1: Remove Old Polluted Variables

First of all, remove the old environment variables and generate the new for clean work and this time we will be using client id and client secret of Falcon KAC which are separate from Falcon Sensor. Run below command to clean the environment:
```powershell
Remove-Variable FALCON_CID -ErrorAction SilentlyContinue

Remove-Variable FALCON_KAC_FULL_PATH -ErrorAction SilentlyContinue

Remove-Variable FALCON_KAC_IMAGE_REPO -ErrorAction SilentlyContinue
```


## Step 2: Regenerate Values Properly

Now, again declare the environment variables. So, hopefully it will not mix with previous values and will be deployed successfully without any error.
```powershell
$env:FALCON_CLIENT_ID="YOUR_CLIENT_ID"

$env:FALCON_CLIENT_SECRET="YOUR_CLIENT_SECRET"
```
Regenerate Values for Falcon KAC by running below commands:
```powershell
$FALCON_CID = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-kac --get-cid"

$FALCON_KAC_IMAGE_FULL_PATH = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-kac --get-image-path"

$FALCON_IMAGE_PULL_TOKEN = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-kac --get-pull-token"
```

Verify the values
```powershell
$FALCON_CID

$FALCON_KAC_IMAGE_FULL_PATH

$FALCON_IMAGE_PULL_TOKEN
```

Slit repo and tag by below commands:
```powershell
$FALCON_KAC_IMAGE_REPO = $FALCON_IMAGE_FULL_PATH.Split(“:”)[0]

$FALCON_KAC_IMAGE_T
```

## Step 3: Create a File for Values and Deploy Falcon KAC

Create file for values by running below command:
```powershell
@"
falcon:
  cid: "$FALCON_CID"
  tags: kac,cloud-lab

image:
  repository: "$FALCON_KAC_IMAGE_REPO"
  tag: "$FALCON_KAC_IMAGE_TAG"
  registryConfigJSON: "$FALCON_IMAGE_PULL_TOKEN"
"@ | Out-File falcon-kac-values.yaml -Encoding utf8
```
Deploy the Falcon KAC using helm:
```powershell
helm upgrade --install falcon-kac crowdstrike/falcon-kac `
  -n falcon-kac --create-namespace `
  -f falcon-kac-values.yaml
```

## Step 4: Verify Deployment

After the deployment is done, verify that the everything is working fine:
```powershell
kubectl get pods -n <falcon-kac-namespace> -o wide
```
Describe pod and check everything is normal:
```powershell
Kubectl describe pod <falcon-kac-pod-name> -n <falcon-kac-namespace>
```
Check the logs of the pod:
```powershell
kubectl logs <falcon-sensor-pod> -n falcon-system
```
