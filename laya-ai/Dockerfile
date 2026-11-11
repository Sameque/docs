# ============================================================
# Laya System One - Imagem "all-in-one" CPU (deps + pesos embutidos)
# Build:  docker build -t laya-serve:0.3.20 .
# Run:    docker run -p 8000:8000 laya-serve:0.3.20
# ============================================================

FROM python:3.12-slim-bookworm

# ---- Etapa 1: ambiente (como root) ------------------------------------
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

# curl: usado pelo HEALTHCHECK
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Torch CPU-only: evita os ~3 GB de wheels CUDA (torch + libs NVIDIA)
# que nunca seriam usados num servidor sem GPU. A 2.14.0 e a versao que
# a imagem oficial do projeto usa (pinada/testada).
ARG TORCH_VERSION=2.14.0
RUN pip install --upgrade pip && \
    pip install "torch==${TORCH_VERSION}" --index-url https://download.pytorch.org/whl/cpu

# Versao pinada para reprodutibilidade. O PyPI pode ter versoes mais novas:
# ao atualizar, ajuste tambem a tag da imagem (README/docker-compose).
ARG LAYA_VERSION=0.3.20
RUN pip install "laya[serve]==${LAYA_VERSION}"

# ---- Etapa 2: usuario nao-root ANTES de baixar os pesos ---------------
# ATENCAO: o UID/GID aqui precisa ser o mesmo do cache mount abaixo.
# A versao antiga usava 'useradd -r', que cria UID de sistema (~999);
# o cache mount com uid=1000 entao falhava com "Permission denied" em
# todas as tentativas de download dos pesos.
RUN groupadd -g 1000 laya && useradd -m -u 1000 -g 1000 -d /opt/laya -s /usr/sbin/nologin laya

USER laya
WORKDIR /opt/laya

ENV HF_HOME=/opt/laya/.cache/huggingface

# ---- Etapa 3: checkpoints com retry + cache BuildKit -------------------
# O repo convaiinnovations/laya ja contem os 3 checkpoints (raiz = english,
# subpastas multilingual/ e typed-decisions/) e e exatamente o que o Router
# do laya-serve carrega por padrao (DEFAULT_MODELS em laya/router.py), entao
# um unico snapshot cobre o preload 100% offline. Nao baixe tambem os repos
# convaiinnovations/laya-multilingual e convaiinnovations/laya-typed-decisions:
# sao copias standalone que o servidor nunca usa.
# HF_HUB_DISABLE_XET=1: o repo usa storage Xet; com hub 2.x + hf_xet o
# download via cache mount pode nao materializar os arquivos grandes em
# blobs/. O caminho classico via CDN sempre grava os blobs completos.
# A verificacao offline ao final falha o build se faltar qualquer arquivo,
# em vez de deixar o servidor quebrar so em runtime.
RUN --mount=type=cache,target=/hf-cache-laya,uid=1000,gid=1000 \
    set -e; \
    mkdir -p ${HF_HOME}; \
    for attempt in 1 2 3 4 5; do \
      echo "=== Tentativa $attempt de 5: convaiinnovations/laya ==="; \
      if HF_HUB_CACHE=/hf-cache-laya HF_HUB_DOWNLOAD_TIMEOUT=60 HF_HUB_DISABLE_XET=1 \
         python -c "from huggingface_hub import snapshot_download; snapshot_download('convaiinnovations/laya')"; then \
        echo "OK: convaiinnovations/laya"; break; \
      else \
        echo "Falha na tentativa $attempt. Aguardando 15s..."; sleep 15; \
        if [ "$attempt" = "5" ]; then echo "ERRO: download falhou 5x" >&2; exit 1; fi; \
      fi; \
    done; \
    mkdir -p ${HF_HOME}/hub; \
    cp -r /hf-cache-laya/models--convaiinnovations--laya ${HF_HOME}/hub/; \
    du -sh ${HF_HOME}/hub/models--convaiinnovations--laya; \
    echo "=== Verificando integridade offline do snapshot ==="; \
    HF_HUB_OFFLINE=1 python -c "from huggingface_hub import snapshot_download; snapshot_download('convaiinnovations/laya'); print('snapshot completo')"; \
    echo "=== Checkpoints prontos em ${HF_HOME} ==="

# ---- Etapa 4: runtime (CPU) ---------------------------------------------
# TORCH_DISABLE_NATIVE_JIT=1: sem ele o torch 2.14 tenta compilar kernels
# Triton na primeira inferencia e falha por falta de compilador C na imagem
# slim (issue #365 do projeto); a imagem oficial usa a mesma variavel.
# OMP_NUM_THREADS: threads de CPU para a inferencia; mantenha <= n. de cores.
ENV LAYA_DEVICE=cpu \
    LAYA_PRELOAD=1 \
    LAYA_HOST=0.0.0.0 \
    LAYA_PORT=8000 \
    LAYA_MODELS="english,multilingual,typed-decisions" \
    OMP_NUM_THREADS=4 \
    TORCH_DISABLE_NATIVE_JIT=1 \
    TRANSFORMERS_OFFLINE=1 \
    HF_HUB_OFFLINE=1

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=300s --retries=3 \
    CMD curl -fsS http://127.0.0.1:${LAYA_PORT}/health || exit 1

CMD ["laya-serve"]
