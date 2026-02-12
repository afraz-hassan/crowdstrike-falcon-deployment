# Falcon Sensor Deployment
Welcome to the falcon sensor deployment. In this file we will be using powershell related commands.

## Step 1: Download Crowdstrike Falcon Script

First of all download the script for the crowdstrike falcon. 

Here is the command to download script:
```powershell
Invoke-WebRequest `
  -Uri "https://github.com/CrowdStrike/falcon-scripts/releases/latest/download/falcon-container-sensor-pull.sh" `
  -OutFile "falcon-container-sensor-pull.sh"

```
Change permission for this script:
```powershell
wsl chmod +x falcon-container-sensor-pull.sh
```
## Step 2: Declare Environment Variables

Set the Environment variables for the falcon sensor deployment:
```powershell
$env:FALCON_CLIENT_ID="YOUR_CLIENT_ID"

$env:FALCON_CLIENT_SECRET="YOUR_CLIENT_SECRET"

$FALCON_CID = wsl bash -c "./falcon-container-sensor-pull.sh -u 

$env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-sensor --get-cid"

$FALCON_IMAGE_FULL_PATH = wsl bash -c "./falcon-container-sensor-pull.sh -u 

$env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-sensor --get-image-path"

$FALCON_IMAGE_PULL_TOKEN = wsl bash -c "./falcon-container-sensor-pull.sh -u 

$env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-sensor --get-pull-token"
```

Verify the variables:
```powershell
$FALCON_CID

$FALCON_IMAGE_FULL_PATH

$FALCON_IMAGE_PULL_TOKEN
```

Split Image Repo and Tag
```powershell
$FALCON_IMAGE_REPO = $FALCON_IMAGE_FULL_PATH.Split(":")[0]

$FALCON_IMAGE_TAG  = $FALCON_IMAGE_FULL_PATH.Split(":")[1]
```

## Step 3: Create a Values File

Create a values file properly by running this command:
```powershell
@"
falcon:
  cid: "$FALCON_CID"
  tags: daemonset,cloud-lab

node:
  image:
    repository: "$FALCON_IMAGE_REPO"
    tag: "$FALCON_IMAGE_TAG"
    registryConfigJSON: "$FALCON_IMAGE_PULL_TOKEN"
"@ | Out-File falcon-values.yaml -Encoding utf8
```
Confirm the contents of file:
```powershell
Get-Content falcon-values.yaml
```

## Step 4: Deploy Falcon Sensor

Deploy falcon sensor using this command:

```powershell
helm upgrade --install falcon-sensor crowdstrike/falcon-sensor `
  -n falcon-system --create-namespace `
  -f falcon-values.yaml
```
## Step 5: Verify Deployment

After the deployment is done, verify that the everything is working fine:
```powershell
kubectl get pods -n falcon-system -o wide
```
Verify Daemon Set:
```powershell
kubectl get daemonset -n falcon-system
```
Keep in mind that the Desired should be equal to your number of nodes.

Describe pod and check everything is normal:
```powershell
Kubectl describe pod <falcon-sensor-pod-name> -n falcon-system
```

Check the logs of the pod:

```powershell
kubectl logs <falcon-sensor-pod> -n falcon-system
```
