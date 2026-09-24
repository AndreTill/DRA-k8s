# DRA GPU Project — полный набор манифестов

## Структура

```
dra-gpu-project/
├── deviceclasses/          # DeviceClass (Physical, MIG, MPS)
├── claims/                 # ResourceClaimTemplate
├── workloads/              # Демо-поды всех типов
├── legacy/                 # Классический device-plugin smoke
├── kustomization.yaml      # Опциональный kustomize
└── README.md
```

## Требования

- Kubernetes ≥ 1.34 (лучше 1.35+)
- Feature gates:
  - DRAExtendedResource
  - DRAConsumableCapacity
  - DRAPartitionableDevices
  - DRAResourceClaimDeviceStatus
  - DRADeviceBindingConditions
- DRA-драйвер `gpu.deckhouse.io` (модуль GPU Deckhouse)
- CDI включён в containerd/CRI-O
- Узлы с NVIDIA H100 (или совместимыми), размеченные под GPU

## Быстрый запуск

```bash
# 1. DeviceClass
kubectl apply -f deviceclasses/

# 2. ResourceClaimTemplate
kubectl apply -f claims/

# 3. Namespaces (создаются в workloads)
kubectl apply -f workloads/00-namespaces.yaml

# 4. Workloads
kubectl apply -f workloads/

# 5. (опционально) legacy smoke
kubectl apply -f legacy/
```

Или одной командой через kustomize:

```bash
kubectl apply -k .
```

## Типы выделения

| Тип нагрузки          | DeviceClass              | Способ запроса                          | Режим     |
|-----------------------|--------------------------|-----------------------------------------|-----------|
| Training / Exclusive  | nvidia-h100-physical     | gpu.deckhouse.io/h100: N                | Physical  |
| Prod Inference        | nvidia-h100-mig-3g40     | gpu.deckhouse.io/h100-mig-40: 1         | MIG       |
| Small AI / Batch      | nvidia-h100-mps          | ResourceClaimTemplate + sharePercent    | MPS       |
| Legacy smoke          | —                        | nvidia.com/gpu: 1                       | Device Plugin |
