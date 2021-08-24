FEDORA_VERSION = 32
DRIVER_VERSION = 460.73.01
IMAGE ?= fedora
CONTAINER_IMAGE ?= ghcr.io/stackhpc/driver
CONTAINER_TAG ?= $(DRIVER_VERSION)-fedora$(FEDORA_VERSION)

validate:
	@if [ -z "$(DRIVER_VERSION)" ]; then \
		echo "DRIVER_VERSION cannot be empty, automatic detection has failed."; \
		exit 1; \
	fi;

build: validate
	echo "Building Docker Image ... "
	docker build \
	  --rm=false \
	  --build-arg FEDORA_VERSION=$(FEDORA_VERSION) \
	  --build-arg DRIVER_VERSION=$(DRIVER_VERSION) \
	  --tag $(CONTAINER_IMAGE):$(CONTAINER_TAG) \
	  fedora

login:
	echo "Username: $(DOCKER_USERNAME)"
	echo "Password: $(DOCKER_PASSWORD)"
	if [ "$(DOCKER_USERNAME)" != "" ]; then \
		echo "$(DOCKER_PASSWORD)" | docker login --username="$(DOCKER_USERNAME)" --password-stdin $(CONTAINER_IMAGE):$(CONTAINER_TAG); \
	fi; \

push: build login
	docker push $(CONTAINER_IMAGE):$(CONTAINER_TAG)

install:
	helm repo add nvidia https://nvidia.github.io/gpu-operator
	helm repo update
	helm upgrade --install gpu-operator --wait nvidia/gpu-operator --namespace kube-system --set driver.repository=ghcr.io/stackhpc --set driver.version=$(DRIVER_VERSION) --set driver.imagePullPolicy="Always" --set psp.enabled=true

restart:
	kubectl rollout restart ds/nvidia-driver-daemonset -n gpu-operator-resources

logs:
	kubectl logs ds/nvidia-driver-daemonset -n gpu-operator-resources -f

pods:
	kubectl get pods -n gpu-operator-resources

.PHONY: validate build login push
