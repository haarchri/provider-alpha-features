# provider-alpha-features

Currently, there is no direct option available to enable alpha features for providers within Upbound. 
However, one workaround is to encapsulate a controllerconfig using a composition and then configure a provider to utilize this controllerconfig.
This enables the provider to run with the desired controllerconfig settings and the enabled alpha features. 

Start validate this approach with the following steps:


Let's begin by obtaining a kubeconfig for our control plane using the Up CLI. Run the following command:
```bash
up ctp kubeconfig get haarchri-test -a upbound --token <token> -f kubeconfig.yaml
```
This command fetches the kubeconfig for the "haarchri-test" control plane using the "upbound" account and the specified token, and it saves the resulting kubeconfig to a file named "kubeconfig.yaml."

```bash
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Upbound
    upbound:
      webIdentity:
        roleARN: arn:aws:iam::609897127049:role/mcp-haarchri-test-controllerconfig
---
apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket
metadata:
  annotations:
    crossplane.io/external-name: mcp-controllerconfig
  name: mcp-controllerconfig
spec:
  managementPolicies: ["Observe"]
  forProvider:
    region: "us-west-2"
EOF
```

This YAML manifest creates a ProviderConfig and Bucket resource in the specified region us-west-2.
The managementPolicies indicate that the observation policy is applied to this resource.
This allows us to test the function and ensure that it operates as expected.

Let's verify if the "observe-only" functionality is working:

```bash
KUBECONFIG=kubeconfig.yaml kubectl describe bucket.s3.aws.upbound.io/mcp-controllerconfig 
Name:         mcp-controllerconfig
Namespace:    
Labels:       <none>
Annotations:  crossplane.io/external-name: mcp-controllerconfig
API Version:  s3.aws.upbound.io/v1beta1
Kind:         Bucket
Metadata:
  Creation Timestamp:  2023-08-11T09:07:08Z
  Finalizers:
    finalizer.managedresource.crossplane.io
  Generation:        2
  Resource Version:  90981
  UID:               1c162241-8fbd-4d07-94f3-ea4b9d951b23
Spec:
  Deletion Policy:  Delete
  For Provider:
    Region:  us-west-2
  Init Provider:
  Management Policies:
    Observe
  Provider Config Ref:
    Name:  default
Status:
  At Provider:
    Acceleration Status:          
    Arn:                          arn:aws:s3:::mcp-controllerconfig
    Bucket Domain Name:           mcp-controllerconfig.s3.amazonaws.com
    Bucket Regional Domain Name:  mcp-controllerconfig.s3.us-west-2.amazonaws.com
    Grant:
      Id:  19b3d90df9849eabfe30c5de1ad145b44d723f74c421ff17b68f4c205866ee94
      Permissions:
        FULL_CONTROL
      Type:               CanonicalUser
      Uri:                
    Hosted Zone Id:       Z3BJ6K6RIION7M
    Id:                   mcp-controllerconfig
    Object Lock Enabled:  false
    Policy:               
    Request Payer:        BucketOwner
    Server Side Encryption Configuration:
      Rule:
        Apply Server Side Encryption By Default:
          Kms Master Key Id:  
          Sse Algorithm:      AES256
        Bucket Key Enabled:   true
    Versioning:
      Enabled:     false
      Mfa Delete:  false
  Conditions:
    Last Transition Time:  2023-08-11T09:07:14Z
    Reason:                ReconcileSuccess
    Status:                True
    Type:                  Synced
    Last Transition Time:  2023-08-11T09:07:18Z
    Reason:                Available
    Status:                True
    Type:                  Ready
Events:                    <none>
```


Please note that this approach uses the existing functionality within Upbound to achieve the desired outcome, and it serves as a workaround for enabling alpha features.
