include .env
.PHONY: help up build down setup-laravel deploy generate-key multiarch-build multiarch-push clean setup-builder build-dev platforms registry-info buildx-status reset-builder

# Default target
help: ## Show this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'

# Variables
PLATFORMS ?= linux/amd64,linux/arm64
DOCKERFILE ?= Dockerfile
BUILDER_NAME ?= multiarch-builder


# Setup multiarch builder
setup-builder: ## Setup Docker buildx builder for multiarch builds with insecure registry support
	@echo "Setting up multiarch builder with insecure registry support..."
	@if ! docker buildx inspect $(BUILDER_NAME) >/dev/null 2>&1; then \
		echo "Creating new buildx builder: $(BUILDER_NAME)"; \
		docker buildx create --name $(BUILDER_NAME) \
			--buildkitd-config-inline="{\"registry\": {\"$(REGISTRY_URL)\": {\"insecure\": true,\"http\": false}}}" \
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

down:
	@spin down

deploy-staging: up
	@if [ ! -f .env.staging ]; then echo "Error: .env.staging file not found"; exit 1; fi
	@echo "Checking if APP_KEY is set in .env.staging..."
	@APP_KEY=$$(grep -E "^APP_KEY=" .env.staging | cut -d'=' -f2- | sed 's/^"//; s/"$$//; s/^'"'"'//; s/'"'"'$$//'); \
	if [ -z "$$APP_KEY" ]; then \
		echo "APP_KEY is empty, generating new key..."; \
		spin exec php php artisan key:generate --env=staging; \
	else \
		echo "APP_KEY is already set, skipping key generation"; \
	fi
	@SPIN_PROJECT_NAME=${DOCKER_STACK_NAME} spin deploy staging

deploy: up
	@if [ ! -f .env.production ]; then echo "Error: .env.production file not found"; exit 1; fi
	@echo "Checking if APP_KEY is set in .env.production..."
	@APP_KEY=$$(grep -E "^APP_KEY=" .env.production | cut -d'=' -f2- | sed 's/^"//; s/"$$//; s/^'"'"'//; s/'"'"'$$//'); \
	if [ -z "$$APP_KEY" ]; then \
		echo "APP_KEY is empty, generating new key..."; \
		spin exec php php artisan key:generate --env=production; \
	else \
		echo "APP_KEY is already set, skipping key generation"; \
	fi
	@SPIN_PROJECT_NAME=${DOCKER_STACK_NAME} spin deploy production

generate-key:
	@spin exec php php artisan key:generate

setup-laravel: generate-key
	@spin exec php php artisan migrate:fresh --seed
	@spin exec php php artisan cache:clear
	@spin exec php php artisan config:clear
