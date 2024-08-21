# Google Summer of Code 2024

**Project:** [Kubernetes Operator for Advanced  Container Checkpoint Management  ](https://summerofcode.withgoogle.com/programs/2024/projects/IqOsdIBz)
**Organization:** [CRIU](https://criu.org)
**Mentor:** [Adrian Reber](https://github.com/adrianreber), [Radostin Stoyanov](https://github.com/rst0git), [Prajwal S N](https://github.com/snprajwal)
**Proposal:** [Parthiba Hazra - CRIU (GSoC 2024)](./Parthiba_Hazra-%20CRIU%20%28GSoC-2024%29.pdf)


----------

## Overview

Container checkpointing is a crucial feature within Kubernetes, facilitating state preservation for containers during operations such as live migration, fault recovery, and more. However, the current implementation lacks essential management functionalities, such as limiting the number of checkpoints and automating their cleanup, which can lead to resource exhaustion and operational challenges.

My project aimed to enhance the [Kubernetes operator](https://github.com/checkpoint-restore/checkpoint-restore-operator) by implementing advanced garbage collection policies tailored to various levels, including global, namespace, pod, and container-specific policies. These enhancements enable fine-grained control over checkpoint management, helping users prevent resource exhaustion and maintain operational efficiency within Kubernetes clusters. 

## Implementation

1.  **GitHub Actions Workflow for CI/CD**:  
    Implemented a GitHub Actions workflow to automate the building and deployment of the `CheckpointRestoreOperator` to a Kind Kubernetes cluster. Verification workflows were added for linting and Go module checks, ensuring code quality and compliance with best practices.
    
2.  **Count-Based Retention Policies**:  
    Introduced global, container, pod, and namespace-level retention policies based on checkpoint count limits. Additionally, a flag `applyPoliciesImmediately` was introduced to allow for the immediate application of these policies.
    
3.  **Storage-Based Retention Policies**:  
    Developed storage/size-based retention policies at the global, container, pod, and namespace levels. These policies allow users to define both the maximum size of individual checkpoints (`maxCheckpointSize`) and the total size of all checkpoints associated with a resource (`maxTotalSizePerContainer`).
    
4.  **Orphan Checkpoint Retention Policy**:  
    Implemented an orphan checkpoint retention policy that gives users control over whether orphaned checkpoints—those associated with deleted resources—should be retained or deleted.
    
5.  **Documentation**:  
    Comprehensive documentation for the checkpoint retention policies was added. [Docs](https://github.com/checkpoint-restore/checkpoint-restore-operator/blob/main/docs/retention_policy.md)

### Challenges

1.  **Policy Priority Mechanism**:  
    So, imagine you’re trying to enforce rules at different levels in Kubernetes (like container, pod, namespace, and global), but there’s a twist—these rules can’t all happen at once without stepping on each other’s toes. Enter the policy priority mechanism! We had to make sure that the most specific rules (like container-level) would win out over broader ones (like global-level). It was like setting up a game of rock-paper-scissors where container beats pod, pod beats namespace, and so on. It took some brain power, but we managed to get these policies to play nice with each other.
    
2. **CI/CD Pipeline Setup with Local Registry**:  
Picture this: you’ve got your operator ready to roll, but now you need to pull the operator’s Docker image from a local registry within your CI environment. Simple, right? Well, it turns out that getting the Kind cluster to cooperate with the local registry was like trying to get a cat to take a bath. I spent way too much time wrestling with the setup before realizing I had been adding an incorrect `containerdConfigPatches` entry in the Kind config!
    
3.  **The Case of the Disappearing Checkpoints**:  
    Here’s a fun one (if you find debugging fun, that is). Our operator was happily managing checkpoints on a local cluster, but when it came to CI, it was like the checkpoints vanished into thin air. The culprit? The `/var/lib/kubelet/checkpoints` directory where Kubernetes stores these checkpoints by default. Although the directory was mounted correctly, our operator just couldn’t see any checkpoints in there. After some head-scratching, it turned out we needed to use extra mounts in the Kind config to get things working. Problem solved—no more disappearing checkpoints, and a lesson learned: sometimes, even when everything _looks_ right, there’s still a sneaky config detail waiting to trip you up!

## Pull requests

[](https://github.com/snprajwal/gsoc-2022/blob/main/README.md#pull-requests)

-   [`checkpoint-restore-operator#3`  Update controller-manager to Allow Operator Access to Checkpoints](https://github.com/checkpoint-restore/checkpoint-restore-operator/pull/3)
-   [`checkpoint-restore-operator#4`  Add CI Workflow for Building and Deploying Operator](https://github.com/checkpoint-restore/checkpoint-restore-operator/pull/4)
-   [`checkpoint-restore-operator#5`  Add count-based checkpoint retention policies](https://github.com/checkpoint-restore/checkpoint-restore-operator/pull/5)
-   [`checkpoint-restore-operator#6`  Add storage/size based policies](https://github.com/checkpoint-restore/checkpoint-restore-operator/pull/6)
-   [`checkpoint-restore-operator#28`  Add orphan checkpoint retention policy](https://github.com/checkpoint-restore/checkpoint-restore-operator/pull/28)

## Future Work
In the future, we plan to implement a `time-based` checkpoint retention policy as outlined in the original proposal. This feature will allow users to set time thresholds for checkpoint retention, automatically deleting checkpoints that exceed the defined age limit. 

## Acknowledgements
I would like to express my deepest gratitude to my mentors, Adrian Reber, Radostin Stoyanov, and Prajwal S N, for their unwavering support and guidance throughout this journey. Their invaluable resources, constructive reviews, and timely advice helped me navigate the project and the GSoC process with confidence. I’m also thankful to the GSoC team for their continuous guidance, making this journey smooth and enriching.