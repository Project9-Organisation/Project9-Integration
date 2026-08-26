## Structure du pipeline CI/CD

Le pipeline automatise la compilation, les tests, l’analyse de qualité, la construction et l’analyse de sécurité des images Docker, puis leur déploiement sur les environnements AWS EKS de staging et de production. Les résultats des déploiements alimentent également les métriques DORA exposées dans Grafana.

Aucune intervention manuelle n'est nécessaire (hormis la création / validation de pull-request)


```mermaid
flowchart TD

    A[Push ou Pull Request] --> B[Déclenchement GitHub Actions]

    B --> C{Branche}

    C -->|Feature branch| D[Pipeline CI]
    C -->|develop| E[Pipeline CI + Déploiement Staging]
    C -->|main| F[Pipeline CI + Release + Déploiement Production]

    subgraph CI["Intégration continue"]
        D1[Build Back-End / Front-End<br/>Gradle / Angular]
        D2[Tests Back-End / Front-End <br/>JUnit / Karma]
        D3[Qualité Back-End / Front-End <br/>SonarQube]
        D4[SAST Back-End / Front-End<br/>Snyk + Dependabot]
        D5[Contrôle du statut global]
        
        D1 --> D2 
        D1 --> D4
        D2 --> D3
        D4 --> D5        
        D3 --> D5

    end

    D --> D1    
    E --> D1
    F --> D1

    D5 --> G{Pipeline valide ?}

    G -->|Non| H[Arrêt du pipeline]
    G -->|Oui - develop| X[Création du tag de version<br/>develop-12ad577]
    G -->|Oui - main| J[Semantic Release<br/>v1.1.0]

    J --> L[Scan de sécurité Trivy]
    X --> L
    L --> M{Vulnérabilités bloquantes ?}

    M -->|Oui| H
    M -->|Non| N[Publication des images dans GitHub Packages]

    N --> O{Environnement cible}

    O -->|develop| P[Déploiement Staging]
    O -->|main| Q[Déploiement Production]

    subgraph CD["Déploiement continu"]
        P --> P1[Mise à jour du contexte EKS Staging]
        P1 --> P2[Déploiement Helm<br/>microcrm-staging]
        P2 --> P3[DAST ZAProxy]

        Q --> Q1[Mise à jour du contexte EKS Production]
        Q1 --> Q2[Déploiement Helm<br/>microcrm-prod]
        Q2 --> Q3[DAST ZAProxy]
    end

    Q3 --> R[Envoi des métriques DORA]
    P3 --> T

    R --> S[Pushgateway]
    S --> T[Prometheus]
    T --> U[Grafana]
```