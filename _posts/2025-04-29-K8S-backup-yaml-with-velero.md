---
title:  K8S 업그레이드시 velero를 통한 YAML 백업
description: >-
K8S 업그레이드시 velero를 통한 YAML 백업
author: jhpark
date: 20250429 16:20:00 +0900
categories: [K8S]
tags: [K8S Upgrade]
pin: true
media_subpath: '/posts/2504291620'
---

# ✅ Velero 설치 및 구성 가이드 (EKS 기준)

---

# 📋 Velero 설치 구조 요약

| 항목 | 내용 |
|------|------|
| 백업 대상 | Kubernetes 리소스 (Deployment, Service, ConfigMap 등) + Persistent Volume |
| 저장 위치 | S3 버킷 (AWS) |
| 클러스터 인증 | IAM Role (Velero Pod에 부여) |
| 볼륨 스냅샷 | AWS EBS Snapshot 연동 가능 |

---

# 📚 설치 순서 상세

---

## Step 1. S3 버킷 준비

- Velero가 백업 데이터를 저장할 S3 버킷 필요

```bash
aws s3api create-bucket --bucket <your-velero-bucket-name> --region <region>
```

✅ S3 버킷은 같은 리전에 있어야 최적화됨.

---

## Step 2. Velero용 IAM Role 생성

**Pod에 부여할 IAM Role**을 만들어야 해. (IRSA 방식)

1. AWS 공식 Velero Policy 복사해서  
2. IAM Policy 생성  
3. ServiceAccount용 IAM Role 생성 (eksctl로 쉽게 가능)

**eksctl 예제:**
```bash
eksctl create iamserviceaccount \
  --cluster <your-cluster-name> \
  --namespace velero \
  --name velero \
  --attach-policy-arn arn:aws:iam::<your-account-id>:policy/VeleroBackupPolicy \
  --approve
```

✅ Velero가 S3와 EBS에 접근할 수 있게 권한을 줘야 함.

---

## Step 3. Velero 설치 (Helm 사용 권장)

### Helm repo 추가
```bash
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm repo update
```

### Velero 설치
```bash
helm install velero vmware-tanzu/velero \
  --namespace velero --create-namespace \
  --set configuration.provider=aws \
  --set configuration.backupStorageLocation.name=default \
  --set configuration.backupStorageLocation.bucket=<your-velero-bucket-name> \
  --set configuration.backupStorageLocation.config.region=<your-region> \
  --set configuration.volumeSnapshotLocation.name=aws \
  --set configuration.volumeSnapshotLocation.config.region=<your-region> \
  --set serviceAccount.name=velero
```

✅ 설치 후 Velero Pod가 떠야 정상.

```bash
kubectl get pods -n velero
```

---

## Step 4. 백업 테스트 (리소스 백업)

### 전체 네임스페이스 백업 예제
```bash
velero backup create full-backup --include-namespaces kube-system,default
```

### 백업 목록 확인
```bash
velero backup get
```

### 복원 예제
```bash
velero restore create --from-backup full-backup
```

✅ 정상적으로 백업/복원이 되면 준비 완료.

---

# 🔥 설치 시 주의사항

| 항목 | 주의 내용 |
|------|-----------|
| S3 버킷 정책 | 퍼블릭 설정 절대 금지 (최소 권한 원칙 적용) |
| IAM Role 권한 | S3, EBS(DescribeVolume, CreateSnapshot 등) 권한 필수 |
| Pod 스케줄링 위치 | velero는 모든 노드에 DaemonSet으로 깔리지 않고 1개만 설치됨 |
| 암호화 | S3 버킷 암호화 설정 권장 (SSE-S3, SSE-KMS) |
| Snapshot | EBS 볼륨 ID에 접근 가능해야 스냅샷 동작 |

---

# ✅ 전체 흐름 요약

```
[S3 Bucket 생성]
    ↓
[IAM Role + Policy 생성]
    ↓
[Helm으로 Velero 설치]
    ↓
[Velero Backup/Restore 테스트]
```

---

# ✅ 한 줄 요약

> **Velero는 Helm으로 EKS에 설치하고, S3 + IAM Role(IRSA) 구성만 제대로 맞추면 리소스와 볼륨 백업을 간편하게 수행할 수 있다.**

---
