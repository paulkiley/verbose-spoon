To create a Kubernetes cluster on VMware Tanzu Kubernetes Grid Integrated (TKGI) that provides logs to Wavefront utilizing Fluentd, and runs on VMware vSphere with NSX-T for networking, you will follow several key steps. This setup involves deploying a Kubernetes cluster through TKGI, configuring logging with Fluentd to collect logs, and then sending those logs to Wavefront for monitoring and analysis.

### Step 1: Prepare Your VMware Environment

1. **vSphere with NSX-T:** Ensure that VMware vSphere is set up with NSX-T for network virtualization. This setup includes configuring NSX-T to work with vSphere, setting up overlay segments for Kubernetes cluster networking, and ensuring that T0 and T1 routers are configured for external access if needed.

2. **Install TKGI:** Install VMware Tanzu Kubernetes Grid Integrated on your vSphere environment. This process involves deploying the TKGI management components and integrating them with your NSX-T setup. Ensure the TKGI CLI is installed on your workstation.

### Step 2: Create a Kubernetes Cluster with TKGI

1. **Create a Cluster:** Use the TKGI CLI to create a Kubernetes cluster. Here’s an example command:

    ```shell
    tkgi create-cluster <cluster-name> --external-hostname <cluster-hostname> --plan <plan-name>
    ```

    Replace `<cluster-name>`, `<cluster-hostname>`, and `<plan-name>` with your desired cluster name, the hostname for external access, and the TKGI plan name you wish to use, respectively.

2. **Get Credentials:** Once the cluster is created, obtain the kubeconfig file for your cluster to interact with it using kubectl:

    ```shell
    tkgi get-credentials <cluster-name>
    ```

3. **Verify Cluster:** Ensure the cluster is running and accessible:

    ```shell
    kubectl get nodes
    ```

### Step 3: Deploy Fluentd to Collect and Forward Logs

1. **Install Fluentd:** You will deploy Fluentd as a DaemonSet so that it runs on every node. The Fluentd configuration needs to be customized to forward logs to the Wavefront proxy.

2. **Configure Fluentd for Wavefront:** You will need a Fluentd configuration that specifies how to forward logs to Wavefront. This involves using the Fluentd Wavefront plugin and setting it to send logs to your Wavefront instance.

    Here is an example DaemonSet definition snippet for Fluentd with the necessary output plugin for Wavefront:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: fluentd-wavefront-config
      namespace: kube-system
    data:
      fluent.conf: |
        <match **>
          @type wavefront
          wavefront_proxy_host <wavefront-proxy-hostname>
          wavefront_proxy_port 2878
          flush_interval 10s
        </match>
    ```

    Replace `<wavefront-proxy-hostname>` with the hostname of your Wavefront proxy. This configuration directs Fluentd to send logs to the Wavefront proxy.

3. **Deploy Fluentd DaemonSet:** Apply the Fluentd DaemonSet and ConfigMap to your cluster:

    ```shell
    kubectl apply -f fluentd-daemonset.yaml
    ```

### Step 4: Deploy the Wavefront Proxy

If you haven’t deployed a Wavefront proxy within your infrastructure, you will need to deploy one. The proxy can be deployed as a Docker container, as a standalone application, or within Kubernetes itself. Ensure it is accessible by Fluentd for log forwarding.

### Step 5: Verify Integration and Monitor Logs

After deploying Fluentd and configuring it to forward logs to Wavefront, verify that logs are being successfully sent to Wavefront:

1. **Check Fluentd Logs:** Ensure Fluentd is running without errors and is forwarding logs as expected.

2. **Wavefront Dashboard:** Log in to your Wavefront dashboard and check for incoming logs. You may need to set up dashboards or alerts within Wavefront to visualize and monitor the logs according to your requirements.

### Note

The exact configurations, especially for Fluentd and the Wavefront proxy, may need adjustments based on your specific environment and versions of the software you are using. Always refer to the latest documentation for each component for the most accurate and detailed guidance.
