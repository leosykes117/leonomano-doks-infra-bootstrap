SHELL := /bin/bash

NAMESPACE := external-secrets
RELEASE   := external-secrets
CHART     := external-secrets/external-secrets

VALUES    := helm/values.yaml
RENDERED  := rendered/external-secrets.yaml
KUSTOMIZE := kustomize
KUSTOMIZE_DIR := .

.PHONY: render apply diff delete clean

## Render Helm chart to plain YAML
render:
	@echo "🔧 Rendering ESO Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	helm template $(RELEASE) $(CHART) \
		-n $(NAMESPACE) \
		--debug --wait --timeout 5m0s \
		-f $(VALUES) \
		| tee $(RENDERED)
	@echo "Rendered: $(RENDERED)"

## Apply patched manifests with Kustomize
apply: render
	@echo "🚀 Applying ESO manifests (Helm + Kustomize)..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) | kubectl apply -f -

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/$(RELEASE) --timeout=180s || true
	kubectl -n $(NAMESPACE) rollout status deploy/$(RELEASE)-console --timeout=180s || true

# Ver estado rápido
status:
	kubectl -n $(NAMESPACE) get pods,svc,pvc -l app.kubernetes.io/instance=$(RELEASE)

# Logs del contenedor principal (cuando ya arranque)
logs:
	kubectl -n $(NAMESPACE) logs deploy/$(RELEASE) -c ESO --tail=200

## Show diff against cluster
diff: render
	@echo "🔍 Diffing ESO manifests..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) \
		| kubectl diff -f - \
		| tee diff.log || true

## Delete ESO resources
delete:
	@echo "🧨 Deleting ESO resources..."
	$(KUSTOMIZE) build $(KUSTOMIZE_DIR) | kubectl delete -f -
	kubectl -n $(NAMESPACE) delete pvc $(RELEASE) --ignore-not-found=true

## Clean generated files
clean:
	@echo "🧹 Cleaning rendered files..."
	rm -f $(RENDERED)
