- 配置

  ​

  - 可参考官网<https://github.com/NVIDIA/k8s-device-plugin#deployment-via-helm>


  - 参考他人博客<https://blog.csdn.net/qq_43684922/article/details/126906146>

    ​

    - 创建两个yaml文件deviceplugin.yaml
      apiVersion: apps/v1
      kind: DaemonSet
      metadata:
        name: nvidia-device-plugin-daemonset
        namespace: kube-system
      spec:
        revisionHistoryLimit: 10
        selector:
      ​    matchLabels:
      ​      name: nvidia-device-plugin-ds
        template:
      ​    metadata:
      ​      creationTimestamp: null
      ​      labels:
      ​        name: nvidia-device-plugin-ds
      ​    spec:
      ​      containers:
      ​      \- args:
      ​        \- --config-file=/etc/nvidia/config
      ​        env:
      ​        \- name: FAIL_ON_INIT_ERROR
      ​          value: "false"
      ​        image: [nvcr.io/nvidia/k8s-device-plugin:v0.13.0](http://nvcr.io/nvidia/k8s-device-plugin:v0.13.0)
      ​        imagePullPolicy: IfNotPresent
      ​        name: nvidia-device-plugin-ctr
      ​        resources: {}
      ​        securityContext:
      ​          allowPrivilegeEscalation: false
      ​          capabilities:
      ​            drop:
      ​            \- ALL
      ​        terminationMessagePath: /dev/termination-log
      ​        terminationMessagePolicy: File
      ​        volumeMounts:
      ​        \- mountPath: /var/lib/kubelet/device-plugins
      ​          name: device-plugin
      ​        \- mountPath: /etc/nvidia
      ​          name: config
      ​      dnsPolicy: ClusterFirst
      ​      priorityClassName: system-node-critical
      ​      restartPolicy: Always
      ​      schedulerName: default-scheduler
      ​      securityContext: {}
      ​      terminationGracePeriodSeconds: 30
      ​      tolerations:
      ​      \- effect: NoSchedule
      ​        key: [nvidia.com/gpu](http://nvidia.com/gpu)
      ​        operator: Exists
      ​      volumes:
      ​      \- hostPath:
      ​          path: /var/lib/kubelet/device-plugins
      ​          type: ""
      ​        name: device-plugin
      ​      \- configMap:
      ​          defaultMode: 420
      ​          name: nvidia-config
      ​        name: config
        updateStrategy:
      ​    rollingUpdate:
      ​      maxUnavailable: 1
      ​    type: RollingUpdate


    - configmap.yaml
      apiVersion: v1
      data:
        config: |
      ​    {
      ​       "version": "v1",
      ​       "sharing": {
      ​         "timeSlicing": {
      ​           "resources": [
      ​             {
      ​               "name": "[nvidia.com/gpu](http://nvidia.com/gpu)",
      ​               "replicas": 100,
      ​             }
      ​           ]
      ​         }
      ​       }
      ​    }
      kind: ConfigMap
      metadata:
        name: nvidia-config
        namespace: kube-system


  - 实验

    ​

    - 两个使用GPU的pod都在node1运行![img](https://api2.mubu.com/v3/document_image/efb05136-e2f9-4fb3-aaba-d5726a31be54-15661181.jpg)
      apiVersion: v1
      kind: Pod
      metadata:
        name: dcgmproftester-1
      spec:
        nodeName: k8s-node1
        restartPolicy: "Never"
        containers:
        \- name: dcgmproftester11
      ​    image: nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
      ​    args: ["--no-dcgm-validation", "-t 1004", "-d 30"]
      ​    resources:
      ​      limits:
      ​         [nvidia.com/gpu:](http://nvidia.com/gpu:) 1
      ​    securityContext:
      ​      capabilities:
      ​        add: ["SYS_ADMIN"]
      \---
      apiVersion: v1
      kind: Pod
      metadata:
        name: dcgmproftester-2
      spec:
        nodeName: k8s-node1
        restartPolicy: "Never"
        containers:
        \- name: dcgmproftester11
      ​    image: nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
      ​    args: ["--no-dcgm-validation", "-t 1004", "-d 30"]
      ​    resources:
      ​      limits:
      ​         [nvidia.com/gpu:](http://nvidia.com/gpu:) 1
      ​    securityContext:
      ​      capabilities:
      ​        add: ["SYS_ADMIN"]


    - 运行结果![img](https://api2.mubu.com/v3/document_image/080d32d0-433c-43bb-8596-884ea76b5b08-15661181.jpg)


  - 错误隔离测试

    ​

    官网上提到这里只是时分，所以不存在错误隔离，错误一定会互相影响，接下来来测试一下情况

    - 两个pod同时在运行的时候，中止其中一个![img](https://api2.mubu.com/v3/document_image/d1b403e1-ccdf-4f2f-b528-90ce89394f94-15661181.jpg)


    - 两个都会停止![img](https://api2.mubu.com/v3/document_image/89bb0e57-ac39-49ec-b8e4-1970d5f24fa8-15661181.jpg)![img](https://api2.mubu.com/v3/document_image/adf4034b-d935-4d48-8d92-8e3d6ecf7316-15661181.jpg)