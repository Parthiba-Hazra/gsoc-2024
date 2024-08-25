# Google Summer of Code 2024

**Project:** [Kubernetes Operator for Advanced  Container Checkpoint Management  ](https://summerofcode.withgoogle.com/programs/2024/projects/IqOsdIBz)

**Organization:** [CRIU](https://criu.org)

**Mentor:** [Adrian Reber](https://github.com/adrianreber), [Radostin Stoyanov](https://github.com/rst0git), [Prajwal S N](https://github.com/snprajwal)

**Proposal:** [Parthiba Hazra - CRIU (GSoC 2024)](https://github.com/Parthiba-Hazra/gsoc-2024/blob/main/Parthiba%20Hazra%20-%20CRIU%20%28GSoC-2024%29.pdf)



----------

## Overview

Container checkpointing was recently introduced as a beta feature in Kubernetes, enabling use-cases such as live migration, fault recovery, and preemptive scheduling. However, the current implementation lacks essential management functionalities, such as limiting the number of checkpoints and automating their cleanup, which can lead to resource exhaustion and operational challenges.

This project aims to develop a [Kubernetes operator](https://github.com/checkpoint-restore/checkpoint-restore-operator) that provides support for this functionality. The checkpoint-restore operator allows users to specify a set of  policies that limit the number of checkpoints  at different levels, including global, namespace, pod, and container-specific policies. This functionality allows users to automatically discard obsolete checkpoints and provides users with fine-grained control for managing container checkpoints in Kubernetes clusters. 

## Implementation Details

1.  **GitHub Actions Workflow for CI/CD**:  
    Developing a GitHub Actions workflow  that automates the building, deployment, and testing of the CheckpointRestoreOperator in a Kubernetes cluster was crucial for verifying the functionality of the features implemented throughout this project. The verification workflows were extended with  additional linting and Go module checks that ensure code quality and compliance with best practices.
    
2.  **Count-Based Retention Policies**:  
    The simplest method for limiting the number of checkpoints is by using a count-based retention policy that allows users to specify a threshold at global, namespace, pod, or container level. This functionality was implemented with custom resource definition (CRD) that allows to specify a limit for the number of checkpoints at a particular level. For example, the following policy would limit the maximum number of checkpoints per namespace, pod and container to 50, 30 and 10, respectively.
    ```yaml
    globalPolicy:
        maxCheckpointsPerNamespace: 50
        maxCheckpointsPerPod: 30
        maxCheckpointsPerContainer: 10
    ```
    This policy is automatically applied when a new checkpoint has been created. , Alternatively, the following flag  can be used to apply the policy with immediate effect:
    ```yaml
        applyPoliciesImmediately: true
    ```
    
3.  **Storage-Based Retention Policies**:  
    While limiting the number of checkpoints is useful, each checkpoint can have different size depending on size of the memory, rootfs diff, and other runtime state the container has. To address this problem, we developed a mechanism that allows users to specify size-based retention policies. These policies can be applied at global, namespace,, pod, and container level, similar to the count-based policy. This functionality allows users to define both the maximum size of individual checkpoints (`maxCheckpointSize`) and the total size of all checkpoints associated with a resource (`maxTotalSizePerContainer`).


    ```yaml
    globalPolicy:
        maxCheckpointSize: 10
        maxTotalSizePerNamespace: 1000
        maxTotalSizePerPod: 500
        maxTotalSizePerContainer: 100
    ```
    
4.  **Orphan Checkpoint Retention Policy**:  
    In addition to count-based and storage-based policies, the checkpoint-restore operator has been extended with orphan checkpoint retention policy that allows users to enable for checkpoints associated with deleted Pods to be automatically removed. This functionality introduces a Pod watcher that handles delete events and allows  users to control  whether orphaned checkpoints—those associated with deleted resources—should be retained or deleted.
    ```yaml
    retainOrphan: false  # Checkpoints will be deleted if the resource is deleted from the cluster.
    ```

### Challenges and Lessons Learned

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
1. **Time-Based Checkpoint Retention Policy:** We will introduce a time-based retention policy that allows users to define time thresholds for checkpoint retention. This policy will automatically delete checkpoints that exceed the specified age limit, providing a more flexible and automated cleanup mechanism.

2. **Extended Storage Parameter Support:** Currently, the storage parameter in the checkpoint retention policy only supports values in megabytes (MB). We plan to extend this functionality to support various units, similar to Kubernetes resource limits. Users will be able to specify storage limits using suffixes like E, P, T, G, M, k, as well as their power-of-two equivalents (Ei, Pi, Ti, Gi, Mi, Ki). This will provide more granular control over storage resource management in the operator. 

## Acknowledgements
I would like to express my deepest gratitude to my mentors, Adrian Reber, Radostin Stoyanov, and Prajwal S N, for their unwavering support and guidance throughout this journey. Their invaluable resources, constructive reviews, and timely advice helped me navigate the project and the GSoC process with confidence. I’m also thankful to the GSoC team for their continuous guidance, making this journey smooth and enriching.
