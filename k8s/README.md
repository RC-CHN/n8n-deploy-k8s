# n8n Kubernetes 部署指南

本目录包含了在 Kubernetes 上部署 n8n (带 Postgres 和 Redis) 所需的所有清单文件。

## 1. 准备工作

在部署之前，**务必**修改配置文件中的敏感信息。

编辑 `k8s/n8n-config.yaml` 文件：
- 修改 `POSTGRES_PASSWORD` 和 `POSTGRES_NON_ROOT_PASSWORD`
- 修改 `ENCRYPTION_KEY` (这对于 n8n 加密凭证至关重要，一旦设定请勿丢失)
- 修改其他您认为需要的配置

## 2. 部署步骤

虽然可以使用 `kubectl apply -f .` 一次性部署，但为了更可控的启动过程，建议按以下顺序执行：

### 第一步：创建命名空间和配置
```bash
# 创建 n8n 命名空间
kubectl apply -f namespace.yaml

# 应用配置文件和初始化脚本
kubectl apply -f n8n-config.yaml
kubectl apply -f n8n-init-script-configmap.yaml
```

### 第二步：部署数据库和缓存
```bash
# 部署 Postgres 和 Redis
kubectl apply -f postgres-deployment.yaml
kubectl apply -f redis-deployment.yaml
```
*等待片刻，确保数据库 Pod 状态为 `Running` 且初始化完成。*

### 第三步：部署 n8n 应用和 Worker
```bash
# 部署主应用 (会自动创建共享 PVC)
kubectl apply -f n8n-deployment.yaml

# 部署 Worker
kubectl apply -f n8n-worker-deployment.yaml
```

## 3. 验证部署

查看所有 Pod 的状态：
```bash
kubectl get pods -n n8n
```

查看服务暴露的 IP (LoadBalancer)：
```bash
kubectl get svc -n n8n
```
找到 `n8n-service` 的 `EXTERNAL-IP`，在浏览器中访问 `http://<EXTERNAL-IP>:5678`。

## 4. 故障排查

如果 n8n 启动失败，通常是因为连不上数据库。查看日志：
```bash
kubectl logs -f -l app=n8n -n n8n
```

如果数据库初始化有问题，查看 Postgres 日志：
```bash
kubectl logs -f -l app=postgres -n n8n
```

## 5. 卸载

```bash
kubectl delete -f .
```
**注意**：这将删除所有资源，但默认情况下 `PersistentVolumeClaim` (PVC) 可能需要手动确认删除，以防止数据意外丢失。