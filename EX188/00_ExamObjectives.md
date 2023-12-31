# EX188 Exam Objectives

## [Implement images using Podman](01_Implement_images_using_Podman.md)

    Understand and use FROM (the concept of a base image) instruction.
    Understand and use RUN instruction.
    Understand and use ADD instruction.
    Understand and use COPY instruction.
    Understand the difference between ADD and COPY instructions.
    Understand and use WORKDIR and USER instructions.
    Understand security-related topics.
    Understand the differences and applicability of CMD vs. ENTRYPOINT instructions.
    Understand ENTRYPOINT instruction with param.
    Understand when and how to expose ports from a Containerfile.
    Understand and use environment variables inside images.
    Understand ENV instruction.
    Understand container volume.
    Mount a host directory as a data volume.
    Understand security and permissions requirements related to this approach.
    Understand the lifecycle and cleanup requirements of this approach.


## [Manage images](03_Manage_Images.md)

    Understand private registry security.
    Interact with many different registries.
    Understand and use image tags
    Push and pull images from and to registries.
    Back up an image with its layers and meta data vs. backup a container state.


## [Run containers locally using Podman](01_Basic_Podman_Commands.md)

    Run containers locally using Podman
    Get container logs.
    Listen to container events on the container host.
    Use Podman inspect.
    Specifying environment parameters.
    Expose public applications.
    Get application logs.
    Inspect running applications.


## [Run multi-container applications with Podman](04_Run_multi-container_applications_with_Podman.md)

    Create application stacks
    Understand container dependencies
    Working with environment variables
    Working with secrets
    Working with volumes
    Working with configuration


## [Troubleshoot containerized applications](05_Troubleshoot_Containerized_Applications.md)
    Understand the description of application resources
    Get application logs
    Inspect running applications
    Connecting to running containers


As with all Red Hat performance-based exams, configurations must persist after reboot without intervention.

During the exam you may be required to work with one or more pre-written applications. You will not be required to modify application code however in some cases you may need to utilize supplied documentation in order to produce a new deployment of a given application.

# References

[EX188 Exam Objectives](https://www.redhat.com/en/services/training/ex188-red-hat-certified-specialist-containers-exam?section=objectives)

[Basic `podman` Commands](01_Basic_Podman_Commands.md)
