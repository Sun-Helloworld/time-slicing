- 配置

  ​

  - 可参考官网<https://github.com/NVIDIA/k8s-device-plugin#deployment-via-helm>


  - 参考他人博客<https://blog.csdn.net/qq_43684922/article/details/126906146>


  - 实验

    ​

    - 两个使用GPU的pod都在node1运行![img](https://api2.mubu.com/v3/document_image/efb05136-e2f9-4fb3-aaba-d5726a31be54-15661181.jpg)
   

    - 运行结果![img](https://api2.mubu.com/v3/document_image/080d32d0-433c-43bb-8596-884ea76b5b08-15661181.jpg)


  - 错误隔离测试

    ​

    官网上提到这里只是时分，所以不存在错误隔离，错误一定会互相影响，接下来来测试一下情况

    - 两个pod同时在运行的时候，中止其中一个![img](https://api2.mubu.com/v3/document_image/d1b403e1-ccdf-4f2f-b528-90ce89394f94-15661181.jpg)


    - 两个都会停止![img](https://api2.mubu.com/v3/document_image/89bb0e57-ac39-49ec-b8e4-1970d5f24fa8-15661181.jpg)![img](https://api2.mubu.com/v3/document_image/adf4034b-d935-4d48-8d92-8e3d6ecf7316-15661181.jpg)
