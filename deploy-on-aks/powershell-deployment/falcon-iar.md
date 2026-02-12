# Falcon Sensor Deployment
Welcome to the falcon image analyzer at runtime deployment. In this file we will be using powershell related commands.


## Step 1: Remove Old Polluted Variables
First of all, remove the old environment variables and generate the new for clean work and this time we will be using new client id and client secret of Falcon IAR which are separate from previous ones. Run below command to clean the environment:
```powershell
Remove-Variable FALCON_CID -ErrorAction SilentlyContinue

Remove-Variable FALCON_IAR_FULL_PATH -ErrorAction SilentlyContinue

Remove-Variable FALCON_IAR_IMAGE_REPO -ErrorAction SilentlyContinue

Remove-Variable FALCON_IAR_IMAGE_TAG -ErrorAction SilentlyContinue

Remove-Variable FALCON_IAR_IMAGE_PULL_TOKEN -ErrorAction SilentlyContinue
```

## Step 2: Regenerate Values Properly
Now, again declare the environment variables. So, hopefully it will not mix with previous values and will be deployed successfully without any error.
```powershell
$env:FALCON_CLIENT_ID="YOUR_CLIENT_ID"

$env:FALCON_CLIENT_SECRET="YOUR_CLIENT_SECRET"
```

Regenerate Values for Falcon KAC by running below commands:
```powershell
$FALCON_CID = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-imageanalyzer --get-cid"

$FALCON_IAR_IMAGE_FULL_PATH = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-imageanalyzer --get-image-path"

$FALCON_IAR_IMAGE_PULL_TOKEN = wsl bash -c "./falcon-container-sensor-pull.sh -u $env:FALCON_CLIENT_ID -s $env:FALCON_CLIENT_SECRET -t falcon-imageanalyzer --get-pull-token"
```

Verify the values
```powershell
$FALCON_CID

$FALCON_KAC_IMAGE_FULL_PATH

$FALCON_IMAGE_PULL_TOKEN
```

Slit repo and tag by below commands:
```powershell
$FALCON_IAR_IMAGE_REPO = $FALCON_IAR_IMAGE_FULL_PATH.Split(":")[0]

$FALCON_IAR_IMAGE_TAG  = $FALCON_IAR_IMAGE_FULL_PATH.Split(":")[1]
```
Define you cluster name and the region. The region is the same as we got in the perquisites (BASE URL). In my case the region is us-2, so I will declare it as us-2. 
```powershell
$FALCON_IAR_CLUSTER_NAME="YOUR_CLUSTER_NAME"

$FALCON_CLOUD_ENV="us-2"
```

## Step 3: Create a File for Values and Deploy Falcon IAR
Create file for values by running below command:
```powershell
@"
deployment:
  enabled: true
crowdstrikeConfig:
  cid: "$FALCON_CID"
  clusterName: "$FALCON_IAR_CLUSTER_NAME"
  clientID: "$env:FALCON_CLIENT_ID"
  clientSecret: "$env:FALCON_CLIENT_SECRET"
  agentRegion: "$FALCON_CLOUD_ENV"
image:
  registryConfigJSON: "$FALCON_IAR_IMAGE_PULL_TOKEN"
  repository: "$FALCON_IAR_IMAGE_REPO"
  tag: "$FALCON_IAR_IMAGE_TAG"
"@ | Out-File falcon-iar-values.yaml -Encoding utf8
```
Deploy the Falcon KAC using helm:
```powershell
helm upgrade --install iar crowdstrike/falcon-image-analyzer `
  -n falcon-image-analyzer --create-namespace `
  -f falcon-iar-values.yaml
```
## Step 4: Verify Deployment
After the deployment is done, verify that the everything is working fine:
```powershell
kubectl get pods -n <falcon-iar-namespace> -o wide
```
Describe pod and check everything is normal:
```powershell
Kubectl describe pod <falcon-iar-pod-name> -n <falcon-iar-namespace>
```
Check the logs of the pod:
```powershell
Kubectl logs <falcon-iar-pod-name> -n <falcon-iar-namespace>
```
