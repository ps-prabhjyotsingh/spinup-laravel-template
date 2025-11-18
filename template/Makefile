.PHONY: help up build generate-key multiarch-build multiarch-push clean setup-builder build-dev platforms registry-info buildx-status reset-builder

# Default target
help: ## Show this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'

# Variables
IMAGE_NAME ?=
TAG ?= latest
PLATFORMS ?= linux/amd64,linux/arm64
DOCKERFILE ?= Dockerfile
BUILDER_NAME ?= multiarch-builder
INSECURE_REGISTRY ?= registry.amaavi.casa


# Setup multiarch builder
setup-builder: ## Setup Docker buildx builder for multiarch builds with insecure registry support
	@echo "Setting up multiarch builder with insecure registry support..."
	@if ! docker buildx inspect $(BUILDER_NAME) >/dev/null 2>&1; then \
		echo "Creating new buildx builder: $(BUILDER_NAME)"; \
		docker buildx create --name $(BUILDER_NAME) \
			--buildkitd-config-inline="{\"registry\": {\"registry.amaavi.casa\": {\"insecure\": true,\"http\": false}}}" \
			--use || { \
			echo "Failed to create $(BUILDER_NAME), trying with default builder"; \
			docker buildx create --name $(BUILDER_NAME) --use; \
		}; \
	else \
		echo "Using existing builder: $(BUILDER_NAME)"; \
		docker buildx use $(BUILDER_NAME); \
	fi
	@docker buildx inspect --bootstrap || echo "Bootstrap failed, but continuing..."

# Build for current platform only
build: ## Build Docker image for current platform
	@if [ -z "$(IMAGE_NAME)" ]; then echo "Error: IMAGE_NAME is not set"; exit 1; fi
	@echo "Building Docker image for current platform..."
	@docker build -f $(DOCKERFILE) -t $(IMAGE_NAME):$(TAG) .

# Multiarch build (build only, don't push)
multiarch-build: setup-builder ## Build multiarch Docker image for amd64 and arm64
	@if [ -z "$(IMAGE_NAME)" ]; then echo "Error: IMAGE_NAME is not set"; exit 1; fi
	@echo "Building multiarch Docker image for platforms: $(PLATFORMS)"
	@docker buildx build \
		--platform $(PLATFORMS) \
		-f $(DOCKERFILE) \
		-t $(IMAGE_NAME):$(TAG) \
		--build-arg BUILDKIT_INLINE_CACHE=1 \
		--provenance=false \
		. || { \
		echo "Build failed, checking if builder exists..."; \
		docker buildx ls; \
		echo "Try running 'make setup-builder' first"; \
		exit 1; \
	}

# Multiarch build and push
multiarch-push: setup-builder ## Build and push multiarch Docker image for amd64 and arm64
	@if [ -z "$(IMAGE_NAME)" ]; then echo "Error: IMAGE_NAME is not set"; exit 1; fi
	@echo "Building and pushing multiarch Docker image for platforms: $(PLATFORMS)"
	@docker buildx build \
		--platform $(PLATFORMS) \
		-f $(DOCKERFILE) \
		-t $(IMAGE_NAME):$(TAG) \
		--build-arg BUILDKIT_INLINE_CACHE=1 \
		--provenance=false \
		--push \
		. || { \
		echo "Push failed, checking if builder exists..."; \
		docker buildx ls; \
		echo "Try running 'make setup-builder' first"; \
		exit 1; \
	}

# Clean up buildx builder and images
clean: ## Clean up buildx builder and dangling images
	@echo "Cleaning up..."
	@docker buildx rm $(BUILDER_NAME) 2>/dev/null || true
	@docker image prune -f

# Development build with dev Dockerfile
build-dev: ## Build development Docker image
	@if [ -z "$(IMAGE_NAME)" ]; then echo "Error: IMAGE_NAME is not set"; exit 1; fi
	@echo "Building development Docker image..."
	@docker build -f Dockerfile.dev -t $(IMAGE_NAME):dev .

# Show available platforms
platforms: ## Show available platforms for multiarch builds
	@echo "Supported platforms:"
	@echo "  linux/amd64 (Intel/AMD 64-bit)"
	@echo "  linux/arm64 (ARM 64-bit)"
	@echo ""
	@echo "Current PLATFORMS variable: $(PLATFORMS)"
	@echo "You can override with: make multiarch-build PLATFORMS=\"linux/amd64,linux/arm64\""

# Show registry configuration
registry-info: ## Show current registry configuration
	@echo "Registry Configuration:"
	@echo "  IMAGE_NAME: $(IMAGE_NAME)"
	@echo "  INSECURE_REGISTRY: $(INSECURE_REGISTRY)"
	@echo ""
	@echo "The builder is configured to allow insecure access to $(INSECURE_REGISTRY)"
	@echo "You can override the registry with: make multiarch-push IMAGE_NAME=\"your-registry/image\""

# Check buildx status and available builders
buildx-status: ## Check Docker buildx status and available builders
	@echo "Docker buildx version:"
	@docker buildx version
	@echo ""
	@echo "Available builders:"
	@docker buildx ls
	@echo ""
	@echo "Current builder:"
	@docker buildx inspect 2>/dev/null || echo "No current builder"

# Reset buildx to default
reset-builder: ## Reset to default Docker builder
	@echo "Resetting to default builder..."
	@docker buildx use default
	@echo "Current builder is now: $$(docker buildx inspect | head -1)"

up:
	@spin up -d
generate-key:
	@spin exec php php artisan key:generate
