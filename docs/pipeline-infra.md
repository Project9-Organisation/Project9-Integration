## Structure du pipeline Infrastructure

L'infrastructure des environnements est provisionnée à l'aide de Terraform via un workflow GitHub Actions dédié, déclenché manuellement.
Le workflow permet de crééer l’infra, mais aussi de modifier l’infra existante, ainsi que de détruire l’infra

Pour lancer un job d'infra :

- Ouvrir le dépôt GitHub et accéder à l'onglet Actions
- Sélectionner le workflow "Trigger infrastructure deployment"
- Cliquer sur "Run workflow"

Choix de l'option

-	plan : Terraform analyse les changements dans le code sans les appliquer
-	apply : Terraform applique les changements dans le code
-	plan-destroy : Terraform analyse la destruction l’appliquer
-	destroy : Terraform détruit l’environnement
 
Choix de l'environnement

- prod / staging

- Cliquer sur "Run workflow"


```mermaid
flowchart TD

    A[Déclenchement manuel<br/>workflow_dispatch]

    A --> B{Choix de l'environnement}

    B -->|staging| C[Environment : staging]
    B -->|prod| D[Environment : prod]

    C --> E{Choix de l'action}
    D --> E

    E -->|plan| F[Terraform<br/>init, fmt, validate]
    E -->|apply| AA[Terraform<br/>init, fmt, validate]
    E -->|destroy| DA[Terraform<br/>init, fmt, validate]
    
    AA --> AB[terraform plan]
    AB --> AC[terraform apply]
    AC --> AD[AWS Services<br/>VPC, Subnets, EKS, ALB, IAM ...]
    AD --> AE[Création du cluster EKS]    
    AE --> AF[Installation des Add-ons EKS Rollout...]    
    AF --> AG[Installation du monitoring]
    AG --> AH[Infrastructure AWS créée / modifiée]

    DA --> DB[terraform destroy<br/>tfplan]    
    DB --> DC[Suppression des Add-ons EKS Rollout...]
    DC --> DD[Suppression du monitoring]
    DD --> DE[Suppression AWS Services<br/>VPC, Subnets, EKS, ALB, IAM ...]
    DE --> DF[Suprression du cluster EKS]    
    DF --> DG[Infrastructure AWS supprimée]
    
```