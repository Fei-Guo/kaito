# KAITO pkg Directory - AI Agent Navigation Guide

## Overview
The `/pkg` directory contains the **core Go packages** that implement KAITO's (Kubernetes AI Toolchain Operator) business logic. This directory serves as the primary codebase for:
- **Kubernetes controllers** (Workspace, RAGEngine, InferenceSet)
- **Model interface and runtime parameter management**
- **GPU SKU handling** (Azure, AWS, Arc)
- **Node provisioning and resource management**
- **Inference and fine-tuning orchestration**
- **Utility functions** for Kubernetes operations

All code in this directory is written in **Go** and integrates with Kubernetes CRDs (Custom Resource Definitions) defined in `/api`.

---

## Directory Structure

### 📁 `/pkg/model/` - Model Interface & Runtime Parameters
Core abstractions for AI model management.

#### **Key Files:**
- **`interface.go`** - Central model interface definitions
  - `Model` interface: Defines methods all models must implement
    - `GetInferenceParameters()` - Returns inference config (GPU, memory, runtime params)
    - `GetTuningParameters()` - Returns fine-tuning config
    - `SupportDistributedInference()` - Multi-node inference capability
    - `SupportTuning()` - Fine-tuning capability
  - `PresetParam` struct: Model metadata and requirements
    - GPU/disk requirements, token limits, image references
    - Runtime parameters for Transformers and vLLM
    - Command generation for inference and tuning
  - `RuntimeContext` struct: Execution context for model deployment
  - `Metadata` struct: Model versioning and registry information

**Use Cases:**
- Add new model: Implement `Model` interface
- Modify runtime behavior: Update `PresetParam` command builders
- Calculate resource requirements: Reference `PresetParam` fields

---

### 📁 `/pkg/workspace/` - Workspace Resource Management
Implements the core Workspace CRD controller and resource managers.

#### **Subdirectories:**

##### `/workspace/controllers/` - Workspace Controller
- **`workspace_controller.go`** - Main reconciliation loop for Workspace CRD
  - Handles Workspace lifecycle (create, update, delete)
  - Orchestrates node provisioning, workload deployment
  - Manages status updates and conditions
- **`workspace_gc_finalizer.go`** - Garbage collection and cleanup logic
- **`workspace_event_handler.go`** - Event handling for dependent resources
- **`metrics.go`** - Prometheus metrics for workspace operations

##### `/workspace/inference/` - Inference Workload Generation
- **`preset_inferences.go`** - Creates StatefulSets for inference workloads
  - Builds containers with model-specific commands
  - Configures volumes, probes, resource requests
  - Handles distributed inference (multi-node via Ray)
- **`template_inference.go`** - Template-based inference for custom models
- **`max_model_len_strategy.go`** - Calculates optimal max-model-len for vLLM
- **`preset_inference_types.go`** - Type definitions for inference configs

##### `/workspace/tuning/` - Fine-Tuning Workload Generation
- **`preset_tuning.go`** - Creates Jobs for fine-tuning workloads
  - Configures LoRA adapters, dataset volumes
  - Sets up training environment and scripts
  - Manages output persistence
- **`preset_tuning_types.go`** - Type definitions for tuning configs

##### `/workspace/manifests/` - Kubernetes Manifest Generation
- **`manifests.go`** - Generates Kubernetes resources
  - Services (ClusterIP, LoadBalancer, headless)
  - StatefulSets for inference
  - Jobs for tuning
  - ConfigMaps for model configuration
  - PersistentVolumeClaims for storage

##### `/workspace/resource/` - Node & NodeClaim Management
- **`node_claim.go`** - NodeClaim lifecycle management (Karpenter)
  - `NodeClaimManager`: Creates/deletes NodeClaims
  - Calculates required node count
  - Tracks NodeClaim readiness
- **`node.go`** - Node resource utilities
  - `NodeManager`: Queries and filters Kubernetes nodes
  - Checks node readiness and GPU availability

##### `/workspace/estimator/` - Node Count Estimation
- **`interfaces.go`** - `NodesEstimator` interface
- **`advancednodesestimator/estimator.go`** - GPU memory-based estimation
  - Calculates nodes needed based on model VRAM requirements
  - Considers KV cache, quantization, tensor parallelism
- **`basicnodesestimator/estimator.go`** - Simple count-based estimation

##### `/workspace/image/` - Container Image Management
- **`puller.go`** - OCI artifact/model weight downloader
  - Uses skopeo to pull container images
  - Extracts model weights to volumes
- **`pusher.go`** - Container image pusher utilities

##### `/workspace/webhooks/` - Admission Webhooks
- **`webhooks.go`** - Validation and mutation webhooks for Workspace CRD
  - Validates workspace specs before creation
  - Enforces resource constraints

---

### 📁 `/pkg/ragengine/` - RAGEngine Controller
Implements the RAGEngine CRD controller for RAG deployments.

#### **Subdirectories:**

##### `/ragengine/controllers/` - RAGEngine Controller
- **`ragengine_controller.go`** - Main reconciliation loop for RAGEngine CRD
  - Deploys RAG inference workloads
  - Manages vector store persistence
- **`ragengine_status.go`** - Status management for RAGEngine
- **`preset_rag.go`** - Preset RAG configurations
- **`ragengine_gc_finalizer.go`** - Cleanup logic

##### `/ragengine/manifests/` - RAGEngine Manifest Generation
- **`manifests.go`** - Generates Kubernetes resources for RAG deployments
  - StatefulSets for RAG engine
  - Services for API access
  - PVCs for vector store persistence

##### `/ragengine/webhooks/` - RAGEngine Webhooks
- **`webhooks.go`** - Validation webhooks for RAGEngine CRD

---

### 📁 `/pkg/inferenceset/` - InferenceSet Controller
Implements the InferenceSet CRD controller for load-balanced inference pools.

#### **Key Files:**
- **`inferenceset_controller.go`** - Main reconciliation loop for InferenceSet CRD
  - Manages multiple Workspace instances
  - Integrates with Gateway API Inference Extension
  - Handles rolling updates and scaling
- **`inferenceset_event_handler.go`** - Event handling for workspaces
- **`metrics.go`** - Prometheus metrics for InferenceSet operations

---

### 📁 `/pkg/sku/` - GPU SKU Definitions
Cloud-specific GPU SKU configurations.

#### **Key Files:**
- **`cloud_sku_handler.go`** - `CloudSKUHandler` interface and generic handler
  - `GPUConfig` struct: SKU metadata (GPU count, memory, model)
  - Methods: `GetSKUs()`, `GetGPUConfig()`, `FilterByGPU()`, `GetBestSKU()`
- **`azure_sku_handler.go`** - Azure GPU SKU definitions
  - List of Standard_N* series VMs (V100, T4, A10, A100, H100, H200)
- **`aws_sku_handler.go`** - AWS GPU instance type definitions
  - p3, p4, p5, g4, g5 instance families
- **`arc_sku_handler.go`** - Azure Arc SKU definitions

**Use Cases:**
- Add new GPU SKU: Update corresponding handler's SKU list
- Query available SKUs: Use `CloudSKUHandler` methods
- SKU selection: Use `GetBestSKU()` for optimal node type

---

### 📁 `/pkg/utils/` - Utility Functions
Shared utilities for Kubernetes operations and common tasks.

#### **Key Files:**

##### Core Utilities:
- **`common.go`** - General-purpose helper functions
  - Command string building (`BuildCmdStr`, `ShellCmd`)
  - Image name construction
  - Label/annotation management
  - HuggingFace URL parsing
- **`controller.go`** - Controller expectation tracking
  - `ControllerExpectations`: Tracks expected resource creations
  - Prevents race conditions in reconciliation loops
- **`crd.go`** - CRD schema utilities
- **`config_parser.go`** - Configuration file parsing
- **`common_preset.go`** - Preset model utilities

##### `/utils/consts/` - Constants
- **`consts.go`** - Application-wide constants
  - Finalizer names
  - Cloud names (Azure, AWS, Arc)
  - Feature flags
  - Port numbers
  - Label/annotation keys
  - Conversion factors (GiB to bytes)

##### `/utils/plugin/` - Model Plugin Registry
- **`plugin.go`** - Model registration system
  - `ModelRegister`: Thread-safe model registry
  - `Registration` struct: Model name + instance
  - `KaitoModelRegister`: Global registry singleton
  - Methods: `Register()`, `MustGet()`, `Has()`, `ListModelNames()`

##### `/utils/resources/` - Resource Utilities
- **`resources.go`** - Generic Kubernetes resource helpers
  - Get/Create/Update operations
  - Owner reference management
- **`nodes.go`** - Node-specific utilities
  - Node filtering by labels
  - GPU availability checks

##### `/utils/nodeclaim/` - NodeClaim Utilities
- **`nodeclaim.go`** - NodeClaim helper functions
  - List NodeClaims for workspace
  - Create NodeClaim manifests
  - Check NodeClaim readiness

##### `/utils/workspace/` - Workspace Utilities
- **`workspace.go`** - Workspace-specific helpers
  - Workspace status checks
  - Condition management

##### `/utils/inferenceset/` - InferenceSet Utilities
- **`inferenceset.go`** - InferenceSet helper functions
  - Status management
  - Workspace creation for InferenceSet

##### `/utils/generator/` - Manifest Generators
- **`generator.go`** - Generic manifest generation utilities
  - ConfigMap generation
  - Volume mount creation
  - Environment variable injection

##### `/utils/test/` - Testing Utilities
- **`test_utils.go`** - Test helper functions
- **`test_model.go`** - Mock model implementations
- **`mock_client.go`** - Mock Kubernetes client

---

### 📁 `/pkg/k8sclient/` - Global Kubernetes Client
Singleton Kubernetes client accessor.

#### **Key Files:**
- **`client.go`** - Global client management
  - `SetGlobalClient()` - Initialize global client
  - `GetGlobalClient()` - Retrieve global client
  - Used by packages that can't inject client dependency

---

### 📁 `/pkg/featuregates/` - Feature Gate Management
Feature flag system for experimental features.

#### **Key Files:**
- **`featuregates.go`** - Feature gate definitions and parsing
  - `FeatureGates` map: Feature name → enabled status
  - Supported features:
    - `vLLM` - vLLM runtime support (default: enabled)
    - `ensureNodeClass` - Ensure NodeClass exists
    - `disableNodeAutoProvisioning` - Disable auto node provisioning
    - `gatewayAPIInferenceExtension` - Gateway API integration
    - `enableInferenceSetController` - InferenceSet controller
  - `ParseAndValidateFeatureGates()` - Parse CLI flags

---

### 📁 `/pkg/version/` - Version Information
Build-time version metadata.

#### **Key Files:**
- **`version.go`** - Version info injection
  - Variables: `Version`, `BuildDate`, `GoVersion`
  - `VersionInfo()` - Formatted version string
  - Populated via `-ldflags` at build time

---

## Common Tasks & File Locations

### 🔧 **Controller Operations**

#### **Workspace Controller Tasks:**
- **Main reconciliation logic**: `workspace/controllers/workspace_controller.go`
- **Add finalizer logic**: `workspace/controllers/workspace_gc_finalizer.go`
- **Add metrics**: `workspace/controllers/metrics.go`
- **Webhook validation**: `workspace/webhooks/webhooks.go`

#### **RAGEngine Controller Tasks:**
- **Main reconciliation logic**: `ragengine/controllers/ragengine_controller.go`
- **Status updates**: `ragengine/controllers/ragengine_status.go`
- **Manifest generation**: `ragengine/manifests/manifests.go`

#### **InferenceSet Controller Tasks:**
- **Main reconciliation logic**: `inferenceset/inferenceset_controller.go`
- **Event handling**: `inferenceset/inferenceset_event_handler.go`
- **Metrics**: `inferenceset/metrics.go`

### 🚀 **Model & Runtime Tasks**

#### **Add New Model:**
1. Create model package in `/presets/workspace/models/<model-name>/`
2. Implement `model.Model` interface in `model.go`
3. Register model using `plugin.KaitoModelRegister.Register()` in `init()`
4. Define `PresetParam` with GPU/memory requirements
5. Add to `supported_models.yaml` in presets

#### **Modify Inference Generation:**
- **Transformers inference**: `workspace/inference/preset_inferences.go`
- **Template inference**: `workspace/inference/template_inference.go`
- **Max model length**: `workspace/inference/max_model_len_strategy.go`

#### **Modify Tuning Generation:**
- **Fine-tuning jobs**: `workspace/tuning/preset_tuning.go`
- **Tuning types**: `workspace/tuning/preset_tuning_types.go`

### 📊 **Resource & SKU Management**

#### **SKU Operations:**
- **Add Azure SKU**: Update `sku/azure_sku_handler.go`
- **Add AWS SKU**: Update `sku/aws_sku_handler.go`
- **SKU selection logic**: `sku/cloud_sku_handler.go`

#### **Node Provisioning:**
- **NodeClaim creation**: `workspace/resource/node_claim.go`
- **Node management**: `workspace/resource/node.go`
- **Node estimation**: `workspace/estimator/advancednodesestimator/estimator.go`

### 🛠️ **Utilities & Helpers**

#### **Common Operations:**
- **Build commands**: `utils/common.go` → `BuildCmdStr()`
- **Parse HF URLs**: `utils/common.go` → `ParseHuggingFaceModelVersion()`
- **Manage expectations**: `utils/controller.go` → `ControllerExpectations`
- **Generate ConfigMaps**: `utils/generator/generator.go`

#### **Feature Gates:**
- **Enable/disable features**: `featuregates/featuregates.go`
- **Check feature status**: Access `featuregates.FeatureGates` map

### 📦 **Manifest Generation**

#### **Workspace Manifests:**
- **Services**: `workspace/manifests/manifests.go` → `GenerateServiceManifest()`
- **StatefulSets**: `workspace/inference/preset_inferences.go` → `CreatePresetInference()`
- **Jobs**: `workspace/tuning/preset_tuning.go` → `CreatePresetTuning()`
- **ConfigMaps**: `utils/generator/generator.go`

#### **RAGEngine Manifests:**
- **All resources**: `ragengine/manifests/manifests.go`

---

## Key Interfaces & Abstractions

### **Model Interface** (`pkg/model/interface.go`)
```go
type Model interface {
    GetInferenceParameters() *PresetParam
    GetTuningParameters() *PresetParam
    SupportDistributedInference() bool
    SupportTuning() bool
}
```

### **NodesEstimator Interface** (`pkg/workspace/estimator/interfaces.go`)
```go
type NodesEstimator interface {
    Name() string
    EstimateNodeCount(ctx, workspace, client) (int32, error)
}
```

### **CloudSKUHandler Interface** (`pkg/sku/cloud_sku_handler.go`)
```go
type CloudSKUHandler interface {
    GetSKUs() []GPUConfig
    GetGPUConfig(sku string) *GPUConfig
    FilterByGPU(gpuModel string, minCount int) []GPUConfig
    GetBestSKU(gpuModel string, count int) *GPUConfig
}
```

---

## Important Design Patterns

### **1. Controller Pattern**
All controllers follow Kubernetes controller-runtime pattern:
- Reconcile loop in `*_controller.go`
- Event handlers in `*_event_handler.go`
- Finalizers in `*_gc_finalizer.go`
- Metrics in `metrics.go`

### **2. Plugin Registry Pattern**
Models register themselves using `init()`:
```go
func init() {
    plugin.KaitoModelRegister.Register(&plugin.Registration{
        Name:     "model-name",
        Instance: &modelInstance{},
    })
}
```

### **3. Factory Pattern**
Manifest generation uses factory functions:
- `CreatePresetInference()` → StatefulSet
- `CreatePresetTuning()` → Job
- `GenerateServiceManifest()` → Service

### **4. Expectation Tracking**
Controllers use `ControllerExpectations` to track async operations:
- Set expectation before creating resource
- Satisfy expectation when resource appears
- Prevents duplicate resource creation

---

## Naming Conventions

### **Files:**
- Controllers: `*_controller.go`
- Tests: `*_test.go`
- Interfaces: `interfaces.go` or `*_types.go`
- Utilities: `*.go` (descriptive names)

### **Structs:**
- Reconciler suffix: `WorkspaceReconciler`, `RAGEngineReconciler`
- Manager suffix: `NodeClaimManager`, `NodeManager`
- Config suffix: `GPUConfig`, `RuntimeContext`

### **Constants:**
- All caps with underscores: `FEATURE_FLAG_VLLM`
- Prefixed by category: `Label*`, `Port*`, `FeatureFlag*`

---

## Testing

### **Test Files:**
- Unit tests: `*_test.go` alongside implementation
- Test utilities: `utils/test/`
- Mock implementations: `utils/test/mock_client.go`, `utils/test/test_model.go`

### **Testing Patterns:**
- Table-driven tests
- Mock Kubernetes clients
- Controller test suites using Ginkgo/Gomega

---

## Dependencies & Imports

### **Key External Dependencies:**
- **controller-runtime**: Kubernetes controller framework
- **client-go**: Kubernetes client library
- **karpenter**: Node provisioning (NodeClaim API)
- **klog**: Structured logging
- **Prometheus client**: Metrics collection

### **Internal Import Patterns:**
```go
// API definitions
kaitov1beta1 "github.com/kaito-project/kaito/api/v1beta1"
kaitov1alpha1 "github.com/kaito-project/kaito/api/v1alpha1"

// Internal packages
pkgmodel "github.com/kaito-project/kaito/pkg/model"
metadata "github.com/kaito-project/kaito/presets/workspace/models"
```

---

## Environment Variables

### **Controller Configuration:**
- `RELEASE_NAMESPACE` - Namespace for KAITO release
- `PRESET_REGISTRY_NAME` - Container registry for model images
- Cloud provider set via CLI flags (Azure, AWS, Arc)

### **Feature Gates:**
Set via CLI flag `--feature-gates=<feature>=<true|false>,...`

---

## Code Navigation Tips for AI Agents

### **1. Find Controller Logic:**
- Workspace: `workspace/controllers/workspace_controller.go`
- RAGEngine: `ragengine/controllers/ragengine_controller.go`
- InferenceSet: `inferenceset/inferenceset_controller.go`

### **2. Understand Model System:**
- Interface: `model/interface.go`
- Registry: `utils/plugin/plugin.go`
- Implementations: `/presets/workspace/models/*/model.go`

### **3. Resource Generation:**
- Inference: `workspace/inference/preset_inferences.go`
- Tuning: `workspace/tuning/preset_tuning.go`
- Manifests: `workspace/manifests/manifests.go`

### **4. SKU & Node Management:**
- SKU definitions: `sku/*_sku_handler.go`
- Node estimation: `workspace/estimator/*/estimator.go`
- NodeClaim management: `workspace/resource/node_claim.go`

### **5. Utilities:**
- Constants: `utils/consts/consts.go`
- Common helpers: `utils/common.go`
- Kubernetes ops: `utils/resources/resources.go`

### **6. Feature Management:**
- Feature gates: `featuregates/featuregates.go`
- Constants: `utils/consts/consts.go` (feature flag names)

---

## Metrics & Observability

### **Prometheus Metrics:**
- Workspace metrics: `workspace/controllers/metrics.go`
- InferenceSet metrics: `inferenceset/metrics.go`
- RAGEngine metrics: `ragengine/controllers/*` (if applicable)

### **Logging:**
- Uses `klog` for structured logging
- Logger instances: `klogger := klog.NewKlogr().WithName("ComponentName")`
- Log levels: `InfoS`, `ErrorS`, `V(level)`

---

## Kubernetes Resources Created

### **By Workspace Controller:**
- **StatefulSets** (inference)
- **Jobs** (fine-tuning)
- **Services** (ClusterIP, LoadBalancer, headless)
- **NodeClaims** (Karpenter)
- **ConfigMaps** (model configuration)
- **PersistentVolumeClaims** (model storage, output)

### **By RAGEngine Controller:**
- **StatefulSets** (RAG engine)
- **Services** (API access)
- **PersistentVolumeClaims** (vector store)

### **By InferenceSet Controller:**
- **Workspaces** (multiple instances)
- **InferencePool** (Gateway API extension)
- **HelmRelease** (Flux CD)

---

## Command-Line Building

### **Command Construction:**
Models specify commands via `PresetParam`:
- **Transformers**: `BuildCmdStr(BaseCommand, Params)`
- **vLLM**: `BuildCmdStr(BaseCommand, ModelRunParams)`
- **Distributed**: `BuildIfElseCmdStr()` for leader/worker selection

Example:
```go
torchCommand := utils.BuildCmdStr("accelerate launch", accelerateParams)
modelCommand := utils.BuildCmdStr("inference_api.py", modelRunParams)
fullCommand := utils.ShellCmd(torchCommand + " " + modelCommand)
```

---

## Error Handling Patterns

### **Standard Error Handling:**
```go
if err := operation(); err != nil {
    return reconcile.Result{}, fmt.Errorf("operation failed: %w", err)
}
```

### **Retry Logic:**
```go
err := retry.RetryOnConflict(retry.DefaultRetry, func() error {
    // Operation that might conflict
})
```

### **Ignore Not Found:**
```go
if err != nil && !apierrors.IsNotFound(err) {
    return reconcile.Result{}, err
}
```

---

## Related Directories

- **`/api`** - CRD definitions (v1alpha1, v1beta1)
- **`/presets`** - Model definitions and runtime code (Python/Go)
- **`/cmd`** - Main entry points for controllers
- **`/config`** - Kubernetes manifests for deployment

---

## Quick Reference: Package Summary

| Package | Purpose | Key Types |
|---------|---------|-----------|
| `model` | Model interface & parameters | `Model`, `PresetParam`, `RuntimeContext` |
| `workspace/controllers` | Workspace reconciliation | `WorkspaceReconciler` |
| `workspace/inference` | Inference workload generation | `CreatePresetInference()` |
| `workspace/tuning` | Tuning workload generation | `CreatePresetTuning()` |
| `workspace/resource` | Node/NodeClaim management | `NodeClaimManager`, `NodeManager` |
| `workspace/estimator` | Node count estimation | `NodesEstimator` |
| `workspace/manifests` | K8s manifest generation | Various generators |
| `ragengine/controllers` | RAGEngine reconciliation | `RAGEngineReconciler` |
| `inferenceset` | InferenceSet reconciliation | `InferenceSetReconciler` |
| `sku` | GPU SKU definitions | `CloudSKUHandler`, `GPUConfig` |
| `utils` | Shared utilities | Various helpers |
| `utils/plugin` | Model registry | `ModelRegister` |
| `utils/consts` | Constants | Feature flags, labels, ports |
| `featuregates` | Feature management | `FeatureGates` map |
| `k8sclient` | Global K8s client | `GetGlobalClient()` |
| `version` | Version info | `VersionInfo()` |

---

*This guide is optimized for AI agent navigation. For human-readable documentation, refer to the main KAITO docs.*
