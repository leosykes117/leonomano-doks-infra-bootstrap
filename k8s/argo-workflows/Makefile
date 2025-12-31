SHELL := /bin/bash

NAMESPACE    ?= argo
ARGO_VERSION ?= v3.7.6
ARGO_SERVER_SVC ?= argo-server
RELEASE   :=

KUSTOMIZE_DIR ?= .
KUSTOMIZE	:= kustomize
RENDERED  := rendered/argo-workflows.yaml
ARGO_INSTALL_URL := https://github.com/argoproj/argo-workflows/releases/download/$(ARGO_VERSION)/install.yaml

.PHONY: render install diff

render:
	@echo "🔧 Rendering Argo Workflows manifests..."
	@mkdir -p $$(dirname "$(RENDERED)")
	curl -fsSL \
  	$(ARGO_INSTALL_URL) \
  	-o "$(RENDERED)"
	@echo "Rendered: $(RENDERED)"

apply: render
	@echo "🚀 Applying Argo Workflows manifests..."
	kubectl create namespace $(NAMESPACE) 2>/dev/null || true
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) | kubectl apply -f -

install: apply
	@echo "Installing Argo Workflows..."
	kubectl -n $(NAMESPACE) rollout status deploy/argo-server --timeout=180s || true
	kubectl -n $(NAMESPACE) rollout status deploy/workflow-controller --timeout=180s || true

diff: render
	@echo "🔍 Diffing Argo Workflows manifests..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) \
		| kubectl diff -f - \
		| tee diff.log || true
