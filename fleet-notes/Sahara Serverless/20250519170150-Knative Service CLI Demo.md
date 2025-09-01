---
"type:": fleet-note
"title:": 20250519170150-Knative Service CLI Demo
id:: 20250519170218  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-05-19T17:02:18  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

```shell
#!/usr/bin/env bash
set -euo pipefail

CLUSTER_NAME="sahara-ai-serverless-platform-research"
CLUSTER_LOCATION="us-central1"

print_usage() {
  cat <<EOF
Usage:
  hive auth
  hive deploy <service-name> -u <user> --image <image>:<version>
  hive status <service-name> -u <user>
EOF
}

# Print the usage if there's no arguments.
if [ $# -lt 1 ]; then
  print_usage
  exit 1
fi

# 
COMMAND="$1"
shift

case "${COMMAND}" in
  auth)
      gcloud container clusters get-credentials \
      "${CLUSTER_NAME}" \
      --location "${CLUSTER_LOCATION}"
      echo "✅ Auth and credentials fetched."
  ;;

  deploy)
    SERVICE_NAME=""
    USER_NAME=""
    IMAGE_SPEC=""

    while [[ $# -gt 0 ]]; do
      case "$1" in
        -u)
          USER_NAME="$2"
          shift 2
          ;;
        --image)
          IMAGE_SPEC="$2"
          shift 2
          ;;
        *)
          if [ -z "${SERVICE_NAME}" ]; then
            SERVICE_NAME="$1"
          else
            echo "Unknown argument: $1"
            exit 1
          fi
          shift
          ;;
      esac
    done

    if [ -z "${SERVICE_NAME}" ] || [ -z "${USER_NAME}" ] || [ -z "${IMAGE_SPEC}" ]; then
      echo "Error: deploy requires <service-name>, -u <user>, --image <image>:<version>"
      print_usage
      exit 1
    fi

    # Check if the namespace already exists
    echo "🔍 Checking namespace '${USER_NAME}'..."
    if kubectl get namespace "${USER_NAME}" &>/dev/null; then
      echo "⚡ Namespace '${USER_NAME}' already exists."
    else
      echo "🆕 Creating namespace '${USER_NAME}'..."
      kubectl create namespace "${USER_NAME}"
      echo "✅ Namespace '${USER_NAME}' created."
    fi

    # Check if the service account already exists
    echo "🔍 Checking ServiceAccount '${USER_NAME}' in namespace '${USER_NAME}'..."
    if kubectl get sa "${USER_NAME}" -n "${USER_NAME}" &>/dev/null; then
      echo "⚡ ServiceAccount '${USER_NAME}' already exists in namespace '${USER_NAME}'."
    else
      echo "🆕 Creating ServiceAccount '${USER_NAME}'..."
      kubectl create sa "${USER_NAME}" -n "${USER_NAME}"
      echo "✅ ServiceAccount '${USER_NAME}' created."
    fi

    # Check if the role already binded
    echo "🔍 Checking RoleBinding '${USER_NAME}' in namespace '${USER_NAME}'..."
    if kubectl get rolebinding "hive-bind-${USER_NAME}" -n "${USER_NAME}" &>/dev/null; then
      echo "⚡ RoleBinding '${USER_NAME}' already exists in namespace '${USER_NAME}'."
    else
      echo "🆕 Creating RoleBinding '${USER_NAME}'..."
      kubectl create rolebinding "hive-bind-${USER_NAME}" \
      --role=knative-serving-core \
      --serviceaccount="${USER_NAME}:${USER_NAME}" \
      -n "${USER_NAME}"
    fi

    # Check if the Knative service already exists. Return 0 if it does.
    if kubectl get ksvc "${SERVICE_NAME}" -n "${USER_NAME}" &>/dev/null; then
      echo "⚡ Knative service '${SERVICE_NAME}' already exists in namespace '${USER_NAME}'."
      exit 0
    fi

    echo "🚀 Deploying Knative service '${SERVICE_NAME}'..."
    kn service create "${SERVICE_NAME}" \
      --port 9080 \
      --image "${IMAGE_SPEC}" \
      --namespace "${USER_NAME}"
    echo "✅ Knative service '${SERVICE_NAME}' deployed in namespace '${USER_NAME}'."
    
    ;;

  status)
    SERVICE_NAME=""
    USER_NAME=""

    while [[ $# -gt 0 ]]; do
      case "$1" in
        -u)
          USER_NAME="$2"
          shift 2
          ;;
        *)
          if [ -z "${SERVICE_NAME}" ]; then
            SERVICE_NAME="$1"
          else
            echo "Unknown argument: $1"
            exit 1
          fi
          shift
          ;;
      esac
    done

    if [ -z "${SERVICE_NAME}" ] || [ -z "${USER_NAME}" ]; then
      echo "Error: status requires <service-name> -u <user>"
      print_usage
      exit 1
    fi

    echo "🔍 Checking status of Knative service '${SERVICE_NAME}' in namespace '${USER_NAME}'..."
    kn service describe "${SERVICE_NAME}" --namespace "${USER_NAME}"
    ;;

  *)
    echo "Unknown command: ${COMMAND}"
    print_usage
    exit 1
    ;;
esac

```

# Reference