# **Using Azure Key Vault with OpenShift**

This guide provides a complete walkthrough for integrating Azure Key Vault with an on-premise OpenShift cluster. By following these steps, you can manage your application secrets centrally in Azure and securely mount them into your OpenShift pods using the Secrets Store CSI Driver.

This process uses a **Service Principal** for authentication, which is a standard and secure method for non-human identities to access Azure resources.

## **Prerequisites**

Before you begin, ensure you have the following tools installed and configured:

* **oc (OpenShift CLI):** Logged into your OpenShift cluster with administrative privileges.  
* **az (Azure CLI):** Logged into the correct Azure account and subscription where your Key Vault resides.  
* **helm (Helm 3):** The package manager for Kubernetes.

## **Step 1: Create and Configure the Azure Service Principal**

The Service Principal is the identity your OpenShift cluster will use to authenticate with Azure.

1. Create the Service Principal  
   Run the following command to create a new Service Principal.  
   az ad sp create-for-rbac \--name "OpenShiftKeyVaultPrincipal"

   The command will output a JSON object. **Immediately copy the appId, password, and tenant values and save them securely.** This is the only time the password (client secret) is displayed.  
   {  
     "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",  
     "displayName": "OpenShiftKeyVaultPrincipal",  
     "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  
     "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  
   }

2. Grant the Service Principal Access to the Key Vault  
   Your Key Vault must be configured to use Azure role-based access control (RBAC). The following command assigns the Key Vault Secrets User role to your new Service Principal, allowing it to read secrets.  
   \# First, get the full resource ID (scope) of your Key Vault  
   KEYVAULT\_SCOPE=$(az keyvault show \--name "your-key-vault-name" \--query "id" \-o tsv)

   \# Now, create the role assignment  
   az role assignment create \\  
     \--role "Key Vault Secrets User" \\  
     \--assignee-object-id $(az ad sp show \--id "YOUR\_APP\_ID" \--query "id" \-o tsv) \\  
     \--assignee-principal-type "ServicePrincipal" \\  
     \--scope $KEYVAULT\_SCOPE  
   **Replace:**  
   * your-key-vault-name with the name of your Azure Key Vault.  
   * YOUR\_APP\_ID with the appId from the previous step.  
3. Create a Test Secret in Azure  
   Create a sample secret in your Key Vault to verify the setup later.  
   az keyvault secret set \--vault-name "your-key-vault-name" \--name "my-test-secret" \--value "ItWorks-On-OpenShift\!"

## **Step 2: Install the Secrets Store CSI Driver on OpenShift**

This component is responsible for fetching secrets and mounting them as volumes. We will use Helm to install the driver and the Azure provider in a single, integrated step.

1. **Add the Required Helm Repositories**  
   helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts  
   helm repo update

2. Install the Driver and Azure Provider  
   This single command installs the main driver and enables the Azure provider plugin, avoiding ownership conflicts between charts.  
   helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver \\  
     \--namespace kube-system \\  
     \--set syncSecret.enabled=true \\  
     \--set azure.enabled=true

3. Verify the Installation  
   Check that the driver and provider pods are running on all your cluster nodes.  
   oc get pods \-n kube-system \-l "app in (secrets-store-csi-driver, csi-secrets-store-provider-azure)"

   You should see pods for both secrets-store-csi-driver and csi-secrets-store-provider-azure in the Running state.

## **Step 3: Configure the Connection in OpenShift**

Now, we'll create the Kubernetes resources that use the Service Principal to connect to your specific Key Vault.

1. Create a Kubernetes Secret with Azure Credentials  
   This secret securely stores the Service Principal's appId and password inside the OpenShift cluster.  
   First, base64 encode your credentials. **Use \-n with echo to avoid adding a newline character.**  
   echo \-n 'YOUR\_APP\_ID' | base64  
   echo \-n 'YOUR\_PASSWORD' | base64

   Create a file named azure-kv-creds.yaml and paste the following, replacing the placeholders with your encoded values.  
   apiVersion: v1  
   kind: Secret  
   metadata:  
     name: azure-kv-creds  
     \# IMPORTANT: Change this to the namespace where your app will run  
     namespace: default  
   type: Opaque  
   data:  
     clientid: "PASTE\_YOUR\_BASE64\_ENCODED\_APPID\_HERE"  
     clientsecret: "PASTE\_YOUR\_BASE64\_ENCODED\_PASSWORD\_HERE"

   Apply the secret to your cluster:  
   oc apply \-f azure-kv-creds.yaml

2. Create the SecretProviderClass  
   This custom resource tells the CSI driver which vault to connect to, which secrets to fetch, and what tenantId to use.  
   Create a file named test-spc.yaml:  
   apiVersion: secrets-store.csi.x-k8s.io/v1  
   kind: SecretProviderClass  
   metadata:  
     name: azure-test-spc  
     \# IMPORTANT: Change this to the namespace where your app will run  
     namespace: default  
   spec:  
     provider: azure  
     parameters:  
       keyvaultName: "your-key-vault-name"  
       usePodIdentity: "false"  
       tenantId: "YOUR\_AZURE\_TENANT\_ID"  
       objects:  |  
         array:  
           \- |  
             objectName: "my-test-secret"  
             objectType: "secret"  
             fileName: "test-secret-file"  
   **Replace:**  
   * your-key-vault-name with your Key Vault's name.  
   * YOUR\_AZURE\_TENANT\_ID with the tenant ID from Step 1\.

Apply it to your cluster:oc apply \-f test-spc.yaml

## **Step 4: Deploy and Verify a Test Pod**

Finally, deploy a sample pod to mount the secret and confirm everything works.

1. Create a Test Pod Manifest  
   Create a file named test-pod.yaml. This pod mounts a volume that uses the SecretProviderClass you just created.  
   apiVersion: v1  
   kind: Pod  
   metadata:  
     name: csi-secret-test-pod  
     \# IMPORTANT: Change this to the namespace where your app will run  
     namespace: default  
   spec:  
     containers:  
       \- name: busybox  
         image: registry.k8s.io/e2e-test-images/busybox:1.29-4  
         command: \["/bin/sleep", "10000"\]  
         volumeMounts:  
           \- name: secrets-store-inline  
             mountPath: "/mnt/secrets-store"  
             readOnly: true  
     volumes:  
       \- name: secrets-store-inline  
         csi:  
           driver: secrets-store.csi.k8s.io  
           readOnly: true  
           volumeAttributes:  
             secretProviderClass: "azure-test-spc"  
           \# This tells the driver which K8s secret holds the credentials  
           nodePublishSecretRef:  
             name: azure-kv-creds

   Apply it to your cluster:  
   oc apply \-f test-pod.yaml

2. Verify the Secret is Mounted  
   Once the pod is in the Running state, get a shell inside it and check for the mounted secret file.  
   \# Get a shell inside the pod  
   oc exec \-it csi-secret-test-pod \-n default \-- sh

   \# Inside the pod, list the files in the mount directory  
   ls /mnt/secrets-store  
   \# Expected output: test-secret-file

   \# Display the content of the file  
   cat /mnt/secrets-store/test-secret-file  
   \# Expected output: ItWorks-On-OpenShift\!

If you see the secret's value, your integration is successful\! You can now adapt the SecretProviderClass and pod volumeMounts for your own applications.