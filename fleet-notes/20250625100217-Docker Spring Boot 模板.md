---
"type:": fleet-note
"title:": 20250625100217-Docker Spring Boot 模板
id:: 20250625100222  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-06-25T10:02:22  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

```dockerfile
FROM eclipse-temurin:17-jdk-jammy  
  
# Set timezone to UTC  
ENV TZ=UTC  
  
# Create non-root user and group  
RUN groupadd --system --gid 1000 appgroup && \  
    useradd --system --uid 1000 --gid appgroup --shell /bin/false --home-dir /app appuser && \  
    mkdir -p /app/jvm-logs /app/dumps && \  
    chown -R appuser:appgroup /app  
  
# Set working directory  
WORKDIR /app  
  
# Configure JVM options with reasonable defaults  
ENV DEFAULT_JAVA_OPTS="\  
-XX:+UseG1GC \  
-XX:MaxRAMPercentage=75.0 \  
-XX:InitiatingHeapOccupancyPercent=45 \  
-XX:MaxGCPauseMillis=200 \  
-XX:+HeapDumpOnOutOfMemoryError \  
-XX:HeapDumpPath=/app/dumps \  
-Xlog:gc*:file=/app/jvm-logs/gc.log:time,uptime,level,tags:filecount=5,filesize=10M \  
"  
  
#ENV JAVA_OPTS=""  
  
# Copy the locally built Spring Boot executable JAR  
# Assumes `mvn clean package` has been run and the JAR is in the target/ directory  
COPY --chown=appuser:appgroup hive-application-start/target/hive-application-start-*.jar app.jar  
  
# Switch to non-root user  
USER appuser:appgroup  
  
# Expose application port (default 8080)  
EXPOSE 8080  
  
## Entrypoint with support for additional JVM parameters and environment variables  
#ENTRYPOINT ["sh", "-c", "exec java $DEFAULT_JAVA_OPTS $JAVA_OPTS -jar /app/app.jar $SPRING_BOOT_OPTS"]  
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

但最好用 `K8s` 注入命令
# Reference