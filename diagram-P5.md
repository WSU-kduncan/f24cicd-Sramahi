```mermaid
graph TD
    A[Hooks.json]
    A --> B[deploy-hook]
    B --> C[Execute deploy.sh]
    C --> D[Set working directory to /home/ec2-user/]

    subgraph SystemD Unit File
        E[webhook.service]
        E --> F[After network.target]
        E --> G[Start Webhook Listener]
        G --> H[Execute webhook with hooks.json]
    end

    subgraph Deployment Script
        I[deploy.sh]
        I --> J[Docker Login]
        J --> K[Pull Image suhayb:v1.0.0]
        K --> L[Stop Running Container]
        L --> M[Remove Old Container]
        M --> N[Run New Container]
    end
