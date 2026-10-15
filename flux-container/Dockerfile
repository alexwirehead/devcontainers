# Образ для работы с ADR и GitOps-репозиториями (from-git/, repos/).
# Версии закреплены; flux, flux-operator, kustomize, kubeconform и yq —
# те же, что в .gitlab-ci.yml. Каждая загрузка сверяется с sha256 из файла
# контрольных сумм релиза: при обрыве сети или подмене файла сборка падает,
# а не ставит битый бинарник. Обновление версии — правка ARG вместе с хешами
# для arm64 и amd64, затем Rebuild.
# Слои кешируются Docker: пока Dockerfile не менялся, Rebuild ничего не качает.
FROM mcr.microsoft.com/devcontainers/base:ubuntu

ARG KUSTOMIZE_VERSION=5.8.3
ARG KUSTOMIZE_SHA256_ARM64=9867b76482cfc6d25b546abc1d2b5a32bd137505aba640649b4b378f64f6d68f
ARG KUSTOMIZE_SHA256_AMD64=cb9e31198d3f63b44848bd0afc4c8efc56e6cfef9c93a4a39a211940e6decd61
ARG KUBECONFORM_VERSION=0.8.0
ARG KUBECONFORM_SHA256_ARM64=1f53fc8e81258197a35e8603054162a5af1de8c5af13746c71ab680d9534ed87
ARG KUBECONFORM_SHA256_AMD64=9bc2bffbf71f261128533edaf912153948b7ff238f9a531ae6d34466ec287883
ARG FLUX_VERSION=2.9.6
ARG FLUX_SHA256_ARM64=6663c154755b732f43dc993d72321f71c2fc1ff0bcb94b2696e0a8638fa562b4
ARG FLUX_SHA256_AMD64=b4d22673e9246cbd628881f1a9ef3b090085dced291e42d804555cee8e8d42c5
ARG FLUX_CRD_SCHEMAS_SHA256=380b713bd28a2c27cfc7710219aae79e1c9ceff9bee50197c023047190532987
ARG FLUX_OPERATOR_VERSION=0.61.0
ARG FLUX_OPERATOR_SHA256_ARM64=499982b3446452b894c9f487a49c18d396b2bdc2b48feb9624ca12aa7eda3001
ARG FLUX_OPERATOR_SHA256_AMD64=5720a6b38e48f950dd97cb06f692695700e0239b28503e1926cc28cf8310145c
ARG FLUX_OPERATOR_CRD_SCHEMAS_SHA256=c062892eeac621948567464ae7688fcafa75c693bdfb170534cb221a56a194d8
ARG YQ_VERSION=4.54.1
ARG YQ_SHA256_ARM64=189088da0c6429ec5178dfaab1a114805f6cab0b61b165ab236efedf1d57a71b
ARG YQ_SHA256_AMD64=8e34fc298390875de416e6a4afcb8cabeceb25d9aa8506c1a2f9353cf702ea5f
ARG PREK_VERSION=0.5.5
ARG PREK_SHA256_ARM64=e7208e5d42daaa8d9167971e7d2d3388d80cce241d1cecea745a6c88d6b6368e
ARG PREK_SHA256_AMD64=57948d4772f4eb58387ce4a6d619ecd2d8d7e9d765eed6a316f5b3f92c86d951
ARG KUBECTL_VERSION=1.37.1
ARG KUBECTL_SHA256_ARM64=ff749f4b78d9c4f1ec87307df9b50119ed819e2094aa9810cb9acffc3286c8c7
ARG KUBECTL_SHA256_AMD64=65691ff77eb6fa44c908b77a1082c9f092c3b9733b5cefabec0d1104890e21a8
ARG HELM_VERSION=4.3.1
ARG HELM_SHA256_ARM64=8cf8f66ecbc252ac8f7f06cca68aba88508ddfde3a6a34feb1678ad1694b4f88
ARG HELM_SHA256_AMD64=0403a88ae20913d11b10a2467b50087721fea7581736da6b9d3992f5357fe54b
ARG SOPS_VERSION=3.13.3
ARG SOPS_SHA256_ARM64=53b0abacd38ef1b12a66d6c100956691b9cefce018d91f81e73ddf7438b94d77
ARG SOPS_SHA256_AMD64=e5bec3346a873ae91d871550f3e698c1aad962aff462a080e40f25fde17fef6b
ARG AGE_VERSION=1.3.2
ARG AGE_SHA256_ARM64=6b8dc4333c53a5a57c9e5834e3a48f92605d7154014cd07269ff3327db5d37f4
ARG AGE_SHA256_AMD64=cbe24006683f8eb669266162894b9a522a1af52f2665fbc63a4bb032ed26ac10
ARG TERRAFORM_VERSION=1.16.6
ARG TERRAFORM_SHA256_ARM64=a750b5a11038f165846b2658c40c88cd7070aa67a61bab431a14ba27928ea9ea
ARG TERRAFORM_SHA256_AMD64=38034b38d03096399a7a1264a14561a8029b2b496fd1f169452656e53968f1b7
# Задаёт BuildKit: arm64 на Apple Silicon, amd64 на Intel.
ARG TARGETARCH

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       ca-certificates curl unzip jq python3 shellcheck postgresql-client \
    && rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    a="${TARGETARCH}"; \
    case "$a" in \
      arm64) p=aarch64; A=ARM64 ;; \
      amd64) p=x86_64; A=AMD64 ;; \
      *) echo "unsupported arch $a"; exit 1 ;; \
    esac; \
    # sha <ИМЯ>: значение ARG <ИМЯ>_SHA256_<ARCH>.
    sha() { eval "echo \"\${$1_SHA256_${A}}\""; }; \
    # fetch <файл> <url> <sha256>: скачать с повторами и сверить хеш.
    fetch() { curl -fsSL --retry 5 --retry-all-errors -o "$1" "$2"; echo "$3  $1" | sha256sum -c -; }; \
    gh=https://github.com; b=/usr/local/bin; t="$(mktemp -d)"; cd "$t"; \
    fetch kustomize.tgz "$gh/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_${a}.tar.gz" "$(sha KUSTOMIZE)"; \
    tar xzf kustomize.tgz -C "$b" kustomize; \
    fetch kubeconform.tgz "$gh/yannh/kubeconform/releases/download/v${KUBECONFORM_VERSION}/kubeconform-linux-${a}.tar.gz" "$(sha KUBECONFORM)"; \
    tar xzf kubeconform.tgz -C "$b" kubeconform; \
    fetch flux.tgz "$gh/fluxcd/flux2/releases/download/v${FLUX_VERSION}/flux_${FLUX_VERSION}_linux_${a}.tar.gz" "$(sha FLUX)"; \
    tar xzf flux.tgz -C "$b" flux; \
    fetch flux-operator.tgz "$gh/controlplaneio-fluxcd/flux-operator/releases/download/v${FLUX_OPERATOR_VERSION}/flux-operator_${FLUX_OPERATOR_VERSION}_linux_${a}.tar.gz" "$(sha FLUX_OPERATOR)"; \
    tar xzf flux-operator.tgz -C "$b" flux-operator; \
    fetch "$b/yq" "$gh/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${a}" "$(sha YQ)"; \
    fetch prek.tgz "$gh/j178/prek/releases/download/v${PREK_VERSION}/prek-${p}-unknown-linux-musl.tar.gz" "$(sha PREK)"; \
    tar xzf prek.tgz --strip-components=1 -C "$b" "prek-${p}-unknown-linux-musl/prek"; \
    fetch "$b/kubectl" "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/${a}/kubectl" "$(sha KUBECTL)"; \
    fetch helm.tgz "https://get.helm.sh/helm-v${HELM_VERSION}-linux-${a}.tar.gz" "$(sha HELM)"; \
    tar xzf helm.tgz --strip-components=1 -C "$b" "linux-${a}/helm"; \
    fetch "$b/sops" "$gh/getsops/sops/releases/download/v${SOPS_VERSION}/sops-v${SOPS_VERSION}.linux.${a}" "$(sha SOPS)"; \
    fetch age.tgz "$gh/FiloSottile/age/releases/download/v${AGE_VERSION}/age-v${AGE_VERSION}-linux-${a}.tar.gz" "$(sha AGE)"; \
    tar xzf age.tgz --strip-components=1 -C "$b" age/age age/age-keygen; \
    fetch terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${a}.zip" "$(sha TERRAFORM)"; \
    unzip -q terraform.zip terraform -d "$b"; \
    chmod 755 "$b"/kustomize "$b"/kubeconform "$b"/flux "$b"/flux-operator "$b"/yq "$b"/prek \
      "$b"/kubectl "$b"/helm "$b"/sops "$b"/age "$b"/age-keygen "$b"/terraform; \
    # CRD-схемы для validate-overlays.sh тех же версий, что flux и flux-operator:
    # без них скрипт качает latest в /tmp при каждом новом контейнере.
    s=/opt/flux-crd-schemas/master-standalone-strict; mkdir -p "$s"; \
    fetch fo-crd.tgz "$gh/controlplaneio-fluxcd/flux-operator/releases/download/v${FLUX_OPERATOR_VERSION}/crd-schemas.tar.gz" "${FLUX_OPERATOR_CRD_SCHEMAS_SHA256}"; \
    fetch flux-crd.tgz "$gh/fluxcd/flux2/releases/download/v${FLUX_VERSION}/crd-schemas.tar.gz" "${FLUX_CRD_SCHEMAS_SHA256}"; \
    tar xzf fo-crd.tgz -C "$s"; tar xzf flux-crd.tgz -C "$s"; \
    cd /; rm -rf "$t"; \
    kustomize version; flux version --client; flux-operator version --client; prek --version; \
    kubectl version --client; helm version --short; sops --disable-version-check --version; \
    age --version; terraform version; yq --version; kubeconform -v

ENV FLUX_SCHEMAS_DIR=/opt/flux-crd-schemas

# Точки монтирования создаются заранее с владельцем vscode: Docker копирует
# владельца из образа в новый именованный том, и chown после старта не нужен.
RUN install -d -o vscode -g vscode \
      /home/vscode/.claude /home/vscode/.agents /home/vscode/.shell-history \
      /home/vscode/.vscode-server \
    && printf '%s\n' 'HISTFILE=~/.shell-history/bash_history' \
       'PROMPT_COMMAND="history -a${PROMPT_COMMAND:+; $PROMPT_COMMAND}"' >> /home/vscode/.bashrc \
    && printf '%s\n' 'HISTFILE=~/.shell-history/zsh_history' >> /home/vscode/.zshenv \
    && chown vscode:vscode /home/vscode/.zshenv
