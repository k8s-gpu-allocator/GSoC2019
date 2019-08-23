# gpu-allocator inspect tool

this is a gpu inspect tool in k8s gpu allocator

## build

```
# go build -o /usr/local/bin/kubectl-inspect-gpu cmd/kubectl-inspect-gpu/*.go
# chmod +x /usr/local/bin/kubectl-inspect-gpu

# cross build
# GOOS=linux GOARCH=amd64 go build -o /usr/local/bin/kubectl-inspect-gpu cmd/kubectl-inspect-gpu/*.go
```

You can refer to the following configuration parameters

| os      | ARCH              | OS version                 |
| ------- | ----------------- |:-------------------------- |
| linux   | 386 / amd64 / arm | >= Linux 2.6               |
| darwin  | 386 / amd64       | OS X (Snow Leopard + Lion) |
| windows | 386 / amd64       | >= Windows 2000            |

## theory

```bash
# inspect
curl 127.0.0.1:12345/gpu-scheduler-extender/inspect
# inspect -d
curl 127.0.0.1:12345/gpu-scheduler-extender/inspect?detail=true
```

## usage

```bash
# kubectl inspect gpu -h
Usage of ./inspect:
  -d    details

# kubectl inspect gpu
NAME                                 ALLGPU  USEDGPU
cn-huhehaote.i-hp32to8ln1xdug4rk401  8       0

# kubectl inspect gpu
gpu policy: static
----------------------------------------
node name: cn-huhehaote.i-hp32to8ln1xdug4rk401
all gpu: 8
used gpu: 0
node type: ecs.gn5-c8g1.14xlarge
gpu topolocy:
      GPU0 GPU1 GPU2 GPU3 GPU4 GPU5 GPU6 GPU7
GPU0     X  PHB  PHB  PHB  PHB  PHB  PHB  PHB
GPU1   PHB    X  PHB  PHB  PHB  PHB  PHB  PHB
GPU2   PHB  PHB    X  PHB  PHB  PHB  PHB  PHB
GPU3   PHB  PHB  PHB    X  PHB  PHB  PHB  PHB
GPU4   PHB  PHB  PHB  PHB    X  PHB  PHB  PHB
GPU5   PHB  PHB  PHB  PHB  PHB    X  PHB  PHB
GPU6   PHB  PHB  PHB  PHB  PHB  PHB    X  PHB
GPU7   PHB  PHB  PHB  PHB  PHB  PHB  PHB    X

Legend:
 X    = Self
 SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
 NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
 PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
 PXB  = Connection traversing multiple PCIe switches (without traversing the PCIe Host Bridge)
 PIX  = Connection traversing a single PCIe switch
 PSB  = Connection traversing a single on-board PCIe switch
 NV#  = Connection traversing a bonded set of # NVLinks

```