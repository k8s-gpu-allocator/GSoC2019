# gpu allocator design

## design goals

When a machine learning task uses n GPU cards, a GPU with strong affinity can be selected to complete the task according to the topology between gpus.

## Non-goals

When there is only one gpu, the GPU topology is not considered.

## device plugin

`nvml` package can get gpu topology info

In Linux machine, the GPU topology information of the machine can be obtained through `nvidia-smi topo -m` .

![image.png](https://cdn.nlark.com/yuque/0/2019/png/394957/1562574603821-31c9f7eb-c5e1-45d0-afa9-83592fa84221.png#align=left&display=inline&height=310&name=image.png&originHeight=619&originWidth=1450&size=89370&status=done&width=725)

here is data structure of the GPU topology

```go

// gpuTopologyType
type gpuTopologyType nvml.P2PLinkType

// gpuTopologyType 字符串描述
func (t gpuTopologyType) String() string {
  return nvml.P2PLinkType(t).String()
}

// gpuTopologyType 字符串简称
func (t gpuTopologyType) Abbreviation() string {
  switch nvml.P2PLinkType(t) {
  case nvml.P2PLinkSameBoard:
    return "PSB"
  case nvml.P2PLinkSingleSwitch:
    return "PIX"
  case nvml.P2PLinkMultiSwitch:
    return "PXB"
  case nvml.P2PLinkHostBridge:
    return "PHB"
  case nvml.P2PLinkSameCPU:
    return "NODE"
  case nvml.P2PLinkCrossCPU:
    return "SYS"
  case nvml.SingleNVLINKLink:
    return "NV1"
  case nvml.TwoNVLINKLinks:
    return "NV2"
  case nvml.ThreeNVLINKLinks:
    return "NV3"
  case nvml.FourNVLINKLinks:
    return "NV4"
  case nvml.P2PLinkUnknown:
  }
  return "N-A"
}

// gpuTopology
type gpuTopology [][]gpuTopologyType
```

here is report gpu topology design:

```go
func getGPUTopologyFromNode(node *v1.Node, devs map[int]*DeviceInfo) [][]uint {
  topology := make([][]uint, len(devs))
  if !utils.IsGPUTopologyNode(node) {
    return topology
  }
  for i := 0; i < len(devs); i++ {
    topology[i] = make([]uint, len(devs))
  }
  for k, v := range node.Annotations {
    if strings.HasPrefix(k, utils.GPU_PRIFX) {
        var gpu1, gpu2 int
        var topoAbbr, topoDesc string
        fmt.Sscanf(k, utils.GPU_PRIFX +  "%s_%d_%d", &topoAbbr,  &gpu1, &gpu2)
        fmt.Sscanf(v, "%s", &topoDesc)
    }
  }
  return topology
}
```

## scheduler extender 

scheduler extender config is as follow:

```json
{
  "kind": "Policy",
  "apiVersion": "v1",
  "extenders": [
    {
      "urlPrefix": "http://127.0.0.1:32743/gpu-scheduler-extender",
      "PrioritizeVerb": "sort",
      "weight": 1,
      "bindVerb":   "bind",
      "enableHttps": false,
      "nodeCacheCapable": true,
      "managedResources": [
        {
          "name": "aliyun.com/gpu",
          "ignoredByScheduler": false
        }
      ],
      "ignorable": false
    }
  ]
}
```

gpu scheduler extedner get gpu topology from node annotation:

```go
func getGPUTopologyFromNode(node *v1.Node, devs map[int]*DeviceInfo) [][]uint {
  topology := make([][]uint, len(devs))
  if !utils.IsGPUTopologyNode(node) {
    return topology
  }
  for i := 0; i < len(devs); i++ {
    topology[i] = make([]uint, len(devs))
  }
  for k, v := range node.Annotations {
    if strings.HasPrefix(k, utils.GPU_PRIFX) {
      var gpu1, gpu2 int
      var topoAbbr, topoDesc string
      fmt.Sscanf(k, utils.GPU_PRIFX +  "%s_%d_%d", &topoAbbr,  &gpu1, &gpu2)
      fmt.Sscanf(v, "%s", &topoDesc)
    }
  }
  return topology
}
```

According to the above ideas, the following figure can be used to describe the whole process briefly.

![](https://cdn.nlark.com/yuque/0/2019/png/394957/1562741756528-d3ea241a-c4ad-4d17-8998-657c31fc45b7.png#align=left&display=inline&height=710&originHeight=710&originWidth=1113&size=0&status=done&width=1113)

