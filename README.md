# AKS-scale-down-mode-for-large-image-caching
AKS-scale-down-mode-for-large-image-caching
src: https://learn.microsoft.com/en-us/azure/aks/scale-down-mode


1.	(for existing nodepool) 
```azurecli
az aks nodepool update --scale-down-mode Deallocate --name f16pool --cluster-name scaledownaks --resource-group scale-down-rg  
(for adding new node pool)
az aks nodepool add --node-count 3 --scale-down-mode Deallocate --node-osdisk-type Managed --name f16pool --cluster-name scaledownaks --resource-group scale-down-rg  
```
2. On scaling down to 0, all the 3 nodes get de-allocated – not deleted 
```azurecli
az aks nodepool scale --node-count 0 --name f16pool --cluster-name scaledownaks --resource-group scale-down-rg  
```
3. Now, scale back to 3; all those 3 nodes had cached image comes back. Pls wait for the node ready state to deploy.  
az aks nodepool scale --node-count 3 --name f16pool --cluster-name scaledownaks --resource-group scale-down-rg

```azurecli
az aks nodepool scale --node-count 3 --name f16pool --cluster-name scaledownaks --resource-group scale-down-rg
```
4. Deploy the same yaml to see the PODS appearing in jiffy; Since the PODS get pulled from local cache it took very few secs to create and run.  

Test background: 
-	Used Contoso container image to repro and noticed the image pull slowness evidently in our setup.
-	Used Docker commands to debug image layers, command ordering etc., – shared a few best practices in specific to reduce layers and sequencing to fix layer caching.
-	Size of the image : 81+ GB ; (>10-15 mins for first time pull)
-	Created 3 node AKS cluster with managed OS disk attached – SKU: Standard F_16 
-	Used LT’s yaml file to deploy PODs- PFA.
-	It got successfully deployed in our nodes and the image started running after 10-12 mins as expected. 
-	Enabled Scale down mode for this node pool SKU and tested a few iterations to confirm the behavior. 
-	At first, it did not show any pods but reconfirmed thru docs that, we must redeploy again to see them in action. 

Result: 
-	We confirmed the image pull slowness has greatly improvised as we leverage local node caching. 
-	Ran 3 iterations and noticed that the PODs are getting created in <8-10 seconds which is good progress. 


![image](https://user-images.githubusercontent.com/61469290/209339013-18ad208e-1637-4bcd-b595-a2a0282ed519.png)

<img width="366" alt="image" src="https://user-images.githubusercontent.com/61469290/209372012-0bb033a3-1eb2-4933-9f24-7c6d63b6c839.png">

![image](https://user-images.githubusercontent.com/61469290/209339020-8400735a-5732-40b4-8e43-a588c24629bc.png)

<img width="546" alt="image" src="https://user-images.githubusercontent.com/61469290/209371769-4d1a0c65-89d0-42ac-a3fd-f510b6b8eda7.png">



