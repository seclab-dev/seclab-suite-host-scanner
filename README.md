# SecLab 主机扫描

SecLab 主机扫描套件镜像，用于 IPv4 主机发现和常见 TCP 端口探测。

## 镜像

```text
guowenju/seclab-host-scanner:0.1.0-alpha.1
```

## 运行示例

```bash
docker network create seclab-suite-network
docker run --rm \
  --name seclab-host-scanner \
  --network seclab-suite-network \
  --cap-add NET_RAW \
  -v seclab-host-scanner-data:/data \
  guowenju/seclab-host-scanner:0.1.0-alpha.1
```

## 本地构建

```bash
./build-image.sh 0.1.0-alpha.1
```

本仓库只维护套件源码和 Docker 镜像。`.slsp` 套件交付包由 `seclab-suites` 仓库统一维护和发布。

在 SecLab 内运行时，后端通过 Suite Runtime SDK 使用 `operation-logs.write` 能力上报扫描提交、终态和删除事件；运行时令牌仅挂载到后端容器。
