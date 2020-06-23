# Helm Install

### Objectives
1. Install Helm version 3

#### 1. Install Helm version 3
```bash
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

#### 2. Verify install 
```bash
helm version --short
```

#### 3. [OPTIONAL] Add bash completion for Helm 

```bash
helm completion bash >> ~/.bash_completion
. ~/.bash_completion
source <(helm completion bash)
```
