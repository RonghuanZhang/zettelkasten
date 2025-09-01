---
"type:": fleet-note
"title:": 20250623105610-Hive Deployment Doc
id:: 20250623105616  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-06-23T10:56:16  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
# Deploy Application

下面是一个基于多阶段构建的 `Dockerfile`，满足以下要求：

* **完全在镜像内部构建**（Maven + JDK 17）
* 运行时只包含最小的 JRE 层
* 暴露端口 `8080`
* 支持通过 `docker run -e` 或 Kubernetes ConfigMap/Secret 注入环境变量
* 使用通配符加载 JAR，忽略版本号

```dockerfile
# -------- Build stage --------
FROM maven:3.8.6-jdk-17-slim AS builder

# 设置工作目录
WORKDIR /workspace

# 只复制 pom 文件，先下载依赖，加快后续 rebuild
COPY pom.xml ./
COPY hive-application-adapter/pom.xml hive-application-adapter/
COPY hive-application-app/pom.xml hive-application-app/
COPY hive-application-client/pom.xml hive-application-client/
COPY hive-application-domain/pom.xml hive-application-domain/
COPY hive-application-infrastructure/pom.xml hive-application-infrastructure/
COPY hive-application-start/pom.xml hive-application-start/

RUN mvn -B dependency:go-offline

# 复制全部代码并编译打包
COPY . .
RUN mvn -B clean package -DskipTests

# -------- Runtime stage --------
FROM eclipse-temurin:17-jre-jammy

# 设置时区（可选）
ENV TZ=Asia/Tokyo

# 应用相关变量（可覆盖）
ENV JAVA_OPTS=""

# 创建应用目录
WORKDIR /app

# 仅复制打包好的 Spring Boot 可执行 JAR
COPY --from=builder /workspace/hive-application-start/target/hive-application-start-*.jar app.jar

# 暴露默认端口（可通过 SPRING_SERVER_PORT 覆盖）
EXPOSE 8080

# 默认启动命令，支持注入 JVM 参数和 Spring 环境变量
ENTRYPOINT [ "sh", "-c", "exec java $JAVA_OPTS -jar /app/app.jar" ]
```

---

## 使用示例

### 本地构建镜像

```bash
docker build -t hive-app:1.0.0 .
```

### 本地运行并注入环境变量

```bash
docker run -d \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:... \
  -e DB_USER=... \
  -e DB_PASS=... \
  hive-app:1.0.0
```

### Kubernetes 示例（Deployment 片段）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hive-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hive-app
  template:
    metadata:
      labels:
        app: hive-app
    spec:
      containers:
        - name: hive-app
          image: your-registry/hive-app:1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              valueFrom:
                configMapKeyRef:
                  name: hive-app-config
                  key: spring.profiles.active
            - name: DB_URL
              valueFrom:
                secretKeyRef:
                  name: hive-app-secret
                  key: db.url
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: hive-app-secret
                  key: db.user
            - name: DB_PASS
              valueFrom:
                secretKeyRef:
                  name: hive-app-secret
                  key: db.pass
          # 如需 JVM 参数，可在 Deployment 里覆盖 JAVA_OPTS
          # env:
          #   - name: JAVA_OPTS
          #     value: "-Xmx512m -Xms256m"
```

---

如此配置即可满足全镜像内构建、按需暴露端口、并通过 Docker 或 Kubernetes 注入环境变量的需求。如有其他特殊需求（比如不同的 JDK 镜像、构建缓存策略等），请告诉我！


# Reference