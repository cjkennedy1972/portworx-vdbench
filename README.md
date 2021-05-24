# portworx-vdbench
Process to run VDBench Raw Volume benchmarks on Portworx Volumes

*This assumes you Already have a working Portworx by Pure Storage cluster in place.
If you do not, visit https://portworx.com for more information.*

By default Kubernetes CSI provisioned volumes are consumed with a file-system in place. However, the timeless benchmark utility, VDBENCH, frequently comes up in conversations with those evaluating data management solutions for Containers.

Here you will find a set of YAML files to provision 3 Portworx Volumes, a manual step to mount them to a host, and a YAML file to bring them up in a Kubernetes managed pod running VDBench. I have also included sample tests in a ConfigMap. This can be edited to change the testing parameters.

## Preparing your environment:
First you will want to download and run vdbench-raw-pvc.yaml. This will create 3 PVCs to use for the benchmarking.
You can verifiy their creation by issuing:

> kubectl get pvc

You should see your three volumes listed.

Next, SSH into the host you want to run the benchmarks from and issue the following commands:

> pxctl volume list

Note the unattached volumes in the list. There should be 3 and these are the volumes you created previously.
In the results of that command, you will want to copy the Name or ID of each volume in the next commands. Here is a sample output:

ID			NAME						SIZE	HA	SHARED	ENCRYPTED	PROXY-VOLUME	IO_PRIORITYSTATUS				SNAP-ENABLED

1054773657008748565	pvc-1a8130f4-98cf-4ef1-9bbb-f293473ccec3	50 GiB	2	no	no		no		HIGH		up 	no

234970879122504832	pvc-3834f418-9a45-42da-94a2-f69f7afb9131	50 GiB	2	no	no		no		HIGH		up 	no

926217724212074340	pvc-3b60289d-f633-4e9e-97f3-c77d32d5508a	50 GiB	2	no	no		no		HIGH		up 	no

966505478598664814	pvc-4a04bb92-a8c8-43f9-ad75-2a7700df6321	10 GiB	1	no	no		no		HIGH		up - attached on 10.10.100.102	no

336691233451103348	pvc-683d136a-b9cf-4dc2-8c0e-dc6f7273cce9	50 GiB	1	no	no		no		HIGH		up - attached on 10.10.100.102	no

985137598051312616	pvc-6ab79f32-def9-44af-b35b-474a38b006a8	5 GiB	1	no	no		no		HIGH		up - attached on 10.10.100.102	no

107592820124132570  pvc-d5e878f6-ce7b-44e2-a1cb-6342859d75c3	10 GiB	1	no	no		no		HIGH		up - attached on 10.10.100.102	no

To mount the unattached volumes to the host you need to copy the ID of the three volumes and issue:

> pxctl host attach 1054773657008748565 && pxctl host attach 234970879122504832 && pxctl host attach 926217724212074340

This will mount the 3 volumes as /dev/pxd/pxd1054773657008748565, etc.
You can issue >ls /dev/pxd to verify the are mounted.
*With the name of the 3 volumes copied to a text editor or some other means, you then log out of the host and proceed.*

## Running the Benchmarks

Next download vdbench-block-local.yaml and edit the device paths in that file to match those on your host. Also replace any instances of nodename with the hostname of the node you mounted the volumes on.

Once you have finished your substitutions, you can issue kubectl create -f vdbench-block-local.yaml to start the pod. Once the pod is created, Exec into it. VDBENCH is located at /vdb/vdbench and the job configuration will be in /templates/vdbench.job. So, to run the benchmark you can run:

> kubectl exec -it pod/vdbench-rawblock -- /bin/bash

This will place you in the root of the pod.

> ./vdb/vdbench -f /templates/vdbench.job

This will kick off the VDBench job and display the results.
