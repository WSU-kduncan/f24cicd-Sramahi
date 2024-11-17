```mermaid
graph TD
    A[Push or Pull Request to Main] --> B[Checkout Repository Code]
    B --> C[Cache Docker Layers]
    C --> D[Log in to DockerHub]
    D --> E[Build Docker Image]
    E --> F[Push Docker Image to DockerHub]
    F --> G[Image Available in DockerHub]
    
