# CST8915 Lab 4: Lab 4 - Introduction to Docker

**Student Name:** Divyang Lodariya 
**Student ID:** 041267824

**Course:** CST8915 Full-stack Cloud-native Development  
**Semester:** Winter 2026

## Demo Video

[Watch Demo Video](https://youtu.be/2TS_HeYItQI?si=zKyVOASobG13fDeH)

---

## Reflection Questions

## Reflection Questions

**i. What are the main differences between a Docker image and a Docker container?**  
A Docker image is a static, read-only template like a blueprint or a packaged snapshot that contains the application code, runtime, libraries, dependencies, and configurations needed to run the software. It is immutable and built from a Dockerfile with layered instructions. A Docker container, on the other hand, is a runnable instance of that image it also adds a thin writable layer on top of the image's read-only layers, allowing the application to execute, make temporary changes, and maintain its own isolated filesystem and processes. The key difference is that images are shareable and reusable while containers are environments that can be started, stopped, or deleted without affecting the underlying image.

**ii. Explain how Docker's layered architecture improves efficiency.**  
Docker's layered architecture stores each instruction in a Dockerfile as a separate, cached read-only layer. When rebuilding an image, Docker reuses unchanged layers from cache and only rebuilds the layers that have changed. This dramatically speeds up builds, reduces disk usage, and minimizes network transfer when pulling or pushing images. It also enables efficient storage and multiple containers from the same image share the same read-only layers, consuming almost no extra space until they write data.

**iii. Why does each container get its own writable layer?**  
Each container receives its own private writable layer so that any changes it makes such as writing files, modifying configuration, or creating temporary data remain isolated and do not affect the base image or other containers running from the same image. This design ensures strong isolation between containers (critical for security and consistency) while preserving efficiency: reads are served from the shared, read-only layers below, and only actual writes go to the small, container-specific layer. Without this, one container's changes could corrupt the image for everyone else or break immutability.

**iv. What are the benefits of using Docker Compose over running containers individually?**  
Docker Compose allows you to define and manage multi-container applications like services, networks, volumes, dependencies, environment variables, ports in a single declarative YAML file, making complex setups reproducible and much easier to understand. Instead of typing long `docker run` commands with many flags, Compose handles startup order, automatic networkin, persistent volumes, scaling, and unified commands like `docker compose up`, `down`, `logs`, and `restart`. This reduces errors, simplifies development workflows, and makes it far easier to replicate the same environment locally, on a VM, or in production.

