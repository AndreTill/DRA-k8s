# DRA GPU Project (community / NVIDIA DRA Driver)

Полный набор манифестов для Dynamic Resource Allocation GPU
на **стандартном community-драйвере NVIDIA**:

- Driver: `gpu.nvidia.com`
- DeviceClasses: `gpu.nvidia.com`, `mig.nvidia.com` (+ кастомные)
- Opaque config: `resource.nvidia.com/v1beta1` / `GpuConfig`
- Extended resource bridge: `nvidia.com/gpu`

Основано на:
- NVIDIA DRA Driver for GPUs (kubernetes-sigs/dra-driver-nvidia-gpu)
- Kubernetes DRA docs
- Статьи Flant (логика слоёв), но API заменён на community

## Структура

```
dra-gpu-project/
├── deviceclasses/     # DeviceClass (Physical, MIG, MPS-oriented)
├── claims/            # ResourceClaimTemplate
├── workloads/         # Демо всех типов
├── legacy/            # Классический device-plugin smoke
├── kustomization.yaml
└── README.md
```

## Требования

- Kubernetes ≥ 1.34 (рекомендуется 1.35+)
- Feature gates: DynamicResourceAllocation, DRAExtendedResource (для мостика)
- Установлен **NVIDIA DRA Driver for GPUs**:
  ```bash
  helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
  helm install nvidia-dra-driver-gpu nvidia/nvidia-dra-driver-gpu \
    --namespace nvidia-dra-driver-gpu --create-namespace \
    --set gpuResourcesEnabledOverride=true
  ```
- CDI включён
- NVIDIA device plugin **отключён** на тех же узлах (конфликт)
- Узлы с GPU (H100 / A100 / и т.д.)

## Быстрый запуск

```bash
kubectl apply -k .
# или по слоям
kubectl apply -f deviceclasses/
kubectl apply -f workloads/00-namespaces.yaml
kubectl apply -f claims/
kubectl apply -f workloads/
```

## Типы выделения

| Тип нагрузки          | DeviceClass / Claim              | Способ запроса                     | Режим        |
|-----------------------|----------------------------------|------------------------------------|--------------|
| Training / Exclusive  | gpu.nvidia.com + nvidia.com/gpu  | limits: nvidia.com/gpu: N          | Physical     |
| Prod Inference MIG    | mig.nvidia.com / custom MIG      | ResourceClaimTemplate              | MIG          |
| Small AI / Batch      | gpu.nvidia.com + GpuConfig MPS    | ResourceClaimTemplate + opaque     | MPS          |
| Time-slicing share    | gpu.nvidia.com + TimeSlicing     | ResourceClaimTemplate + opaque     | TimeSlicing  |
| Legacy smoke          | —                                | nvidia.com/gpu: 1                  | Device Plugin|
