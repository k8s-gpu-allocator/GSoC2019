# Summary of GSoC 2019

Kubernetes with hardware devices topology awareness at node level

Student: Junjun Li(junjunli666@gmail.com)

Mentor: Mentors: Harry Zhang (@resouer), Kai Zhang (@wsxiaozhang), Jian He (@jian-he)

Description: We would like to propose a improvement on current Kubernetes topology manager to become aware of generic hardware device topology at node level, so Deep Learning training can be improved significantly due to data inter-connection between NVIDIA GPU devices on the node.
Recommended Skills: Kubernetes, Golang, basic Machine Learning training experience

# 1.  Overview

I am very happy to have participated in gsoc in the past three months.
I learned a lot of knowledge by communicating with my mentor every week. I'm very excited for it.
Now that gsoc 2019 is almost over, I have completed most of the work on this project. The specific work is as follows:

- the design and implementation of gpu device plugin

- the design and implementation of gpu scheduler extender

- the design and implementation of kubectl inspect gpu

- the design and implementation of multiple allocation strategies

- the design and implementation of simple strategy

# 2. Documents


- [kubectl-inspect-gpu](./inspect.md)

- [gpu-allocator](./gpu-allocator.md)

# 3.Tests

## 3.1 test job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: test-gpu-topology
spec:
  template:
    spec:
      containers:
      - name: test-gpu-topology
        image: registry.cn-hangzhou.aliyuncs.com/konnase/horovod-benchmark:ubuntu1804-cuda10.0-cudnn7.6.0.64-1-nccl2.4.7-1-py36-f3d3b95-horovod-0.16.4-tf1.14.0-torch1.1.0-mxnet1.4.1-test3
        imagePullPolicy: IfNotPresent
        resources:
            limits:
              aliyun.com/gpu: 2
        command: ["sh", "./launch-example.sh", "1", "2"]  # 1 表示机器的数量， 4表示gpu的数量
      restartPolicy: Never
```

![test job result](https://cdn.nlark.com/yuque/0/2019/png/394957/1564480897020-0ba9478c-08e6-4088-bf70-5eff36984178.png?x-oss-process=image/resize,w_1492)

## 3.2 test simple policy

The default installation is the simple policy, and the following actions are used to validate the simple policy:

show `kubectl get pod`

![image.png](https://cdn.nlark.com/yuque/0/2019/png/394957/1566353135840-f5e76efd-1417-4bba-be13-9756dd670813.png#align=left&display=inline&height=45&name=image.png&originHeight=90&originWidth=1502&size=48238&status=done&width=751)

show `kubectl describe pod`


![image.png](https://cdn.nlark.com/yuque/0/2019/png/394957/1566353241434-b2935ec8-b368-456f-862d-cf1d4bb89a2e.png#align=left&display=inline&height=279&name=image.png&originHeight=558&originWidth=1646&size=183454&status=done&width=823)

The above situation shows that the dispatch is successful!

show the reported GPU TOPO structure:

![image.png](https://cdn.nlark.com/yuque/0/2019/png/394957/1566353328460-6de2f57a-5890-407d-abe5-93e22b9e70e4.png#align=left&display=inline&height=444&name=image.png&originHeight=888&originWidth=1572&size=289655&status=done&width=786)

that means the success of Topo structure reporting.

show log logs:

![image.png](https://cdn.nlark.com/yuque/0/2019/png/394957/1564294376195-f6917e1b-9798-4c88-b525-9e8c35efb73a.png#align=left&display=inline&height=531&name=image.png&originHeight=1062&originWidth=1052&size=332388&status=done&width=526)

The following results indicate the completion of task training.

