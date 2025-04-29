---
title:  K8S 업그레이드시 velero를 통한 configMap, secrets 백업
description: >-
K8S 업그레이드시 velero를 통한 YAML 백업
author: jhpark
date: 20250429 16:25:00 +0900
categories: [K8S]
tags: [K8S Upgrade]
pin: true
media_subpath: '/posts/2504291625'
---


# ✅ Secrets, ConfigMap 백업/복구 기본 개념

| 항목 | 설명 |
|------|------|
| ConfigMap | Kubernetes 리소스, 설정값 저장용 (평문 데이터) |
| Secret | 민감한 데이터(패스워드, 토큰 등) 저장 (Base64 인코딩) |
| 기본 Velero 설정 | ConfigMap + Secret 둘 다 백업됨 |

✅ **Velero는 기본적으로 Kubernetes의 모든 리소스 (Deployment, Pod, Service, ConfigMap, Secret 등)**  
✅ **전체를 YAML 형태로 S3에 저장해.**

---

# 📚 실전 포인트

## 1. ConfigMap
- 특별한 설정 없이 **Velero가 자동으로 백업/복원 가능**
- 평문 데이터라서 별 이슈 없음

✅ 그냥 Velero Backup에 포함되기만 하면 된다.

---

## 2. Secret
- 기본적으로 백업은 되지만, **보안 이슈**가 있다.
- S3에 저장될 때 **Base64 인코딩된 형태로 저장된다. (암호화 X)**
- 그래서 **S3 버킷은 암호화 + 퍼블릭 접근 차단**을 반드시 해야 함.

🔹 Velero는 **Secrets를 따로 암호화해서 저장하지 않는다.**  
**(S3 Bucket Level 암호화가 필수야.)**

---

# 🛠️ S3 암호화 강제 설정 예시

버킷을 만들 때:

```bash
aws s3api put-bucket-encryption \
  --bucket <your-velero-bucket-name> \
  --server-side-encryption-configuration '{
      "Rules": [
          {
              "ApplyServerSideEncryptionByDefault": {
                  "SSEAlgorithm": "AES256"
              }
          }
      ]
  }'
```

✅ 또는 KMS를 써서 더 강력하게 관리할 수도 있어.

---

# 🔥 실전에서 놓치기 쉬운 부분

| 항목 | 설명 |
|------|------|
| Secret 안에 있는 데이터는 Base64 인코딩되어 저장될 뿐, 암호화되어 저장되지는 않음 | S3 암호화 강제 |
| Velero Backup 시 Secret을 제외하거나 필터링할 수도 있음 | 필요시 `--exclude-resources secrets` 옵션 사용 |
| 복원 시 Secrets도 함께 복구되며, 별도 설정은 필요 없음 | 다만 다른 클러스터로 복원할 때 네임스페이스 맞추기 주의 |

---

# ✅ 실전 Velero 백업 명령 예시 (Secrets 포함)

전체 백업 (Secrets 포함):
```bash
velero backup create full-backup --include-namespaces kube-system,default
```

Secrets 제외하고 백업:
```bash
velero backup create backup-without-secrets --exclude-resources secrets
```

특정 Secret만 백업:
```bash
velero backup create only-secrets --include-resources secrets
```

---

# 📋 복원(Restoration) 시 주의사항

- Secret이 존재하는 네임스페이스가 먼저 있어야 복원이 정상 진행된다.
- Secret 복원 시 덮어쓰기(overwrite) 여부를 명시할 수 있다.
- `velero restore` 시 옵션 추가 가능:
  - `--restore-volumes=false` (볼륨 복원 제외)
  - `--include-resources secrets`

✅ 예시:
```bash
velero restore create restore-secrets --from-backup full-backup --include-resources secrets
```

---

# ✅ 한 줄 요약

> **Velero는 기본적으로 ConfigMap과 Secret을 함께 백업한다. 하지만 Secret은 암호화되지 않으므로 반드시 S3 암호화와 접근제어를 적용해야 하고, 복원 시 네임스페이스를 정확히 맞춰야 한다.**

---

# 🚀 추가로 바로 도와줄 수 있는 것

| 항목 | 설명 |
|------|------|
| Secret만 별도로 백업/복구하는 스크립트 예제 | ✔️ |
| S3 버킷 정책 예시 (암호화 + 퍼블릭 차단) | ✔️ |
| Velero 백업/복구 실습 시나리오 문서 | ✔️ |

---

