SHELL := /bin/bash

NAMESPACE := argo
RELEASE   := minio
CHART     := oci://registry-1.docker.io/bitnamicharts/minio

VALUES    := helm/values.yaml
RENDERED  := rendered/minio.yaml
KUSTOMIZE := kustomize
KUSTOMIZE_DIR := .

.PHONY: render apply diff delete clean

## Render Helm chart to plain YAML
render:
	@echo "🔧 Rendering MinIO Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	helm template $(RELEASE) $(CHART) \
	  -n $(NAMESPACE) \
		--debug --wait --timeout 5m0s \
	  -f $(VALUES) \
	  | tee $(RENDERED)
	@echo "Rendered: $(RENDERED)"

## Apply patched manifests with Kustomize
apply: render
	@echo "🚀 Applying MinIO manifests (Helm + Kustomize)..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) | kubectl apply -f -

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/$(RELEASE) --timeout=180s || true
	kubectl -n $(NAMESPACE) rollout status deploy/$(RELEASE)-console --timeout=180s || true

# Ver estado rápido
status:
	kubectl -n $(NAMESPACE) get pods,svc,pvc -l app.kubernetes.io/instance=$(RELEASE)

# Logs del contenedor principal (cuando ya arranque)
logs:
	kubectl -n $(NAMESPACE) logs deploy/$(RELEASE) -c minio --tail=200

## Show diff against cluster
diff: render
	@echo "🔍 Diffing MinIO manifests..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) \
		| kubectl diff -f - \
		| tee diff.log || true

## Delete MinIO resources
delete:
	@echo "🧨 Deleting MinIO resources..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) | kubectl delete -f -
	kubectl -n $(NAMESPACE) delete pvc $(RELEASE) --ignore-not-found=true

## Clean generated files
clean:
	@echo "🧹 Cleaning rendered files..."
	rm -f $(RENDERED)
