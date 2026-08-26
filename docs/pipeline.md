## Structure du pipeline CI/CD

Le pipeline automatise la compilation, les tests, l’analyse de qualité, la construction et l’analyse de sécurité des images Docker, puis leur déploiement sur les environnements d'intégration (staging) et de production. Les résultats des déploiements alimentent également les métriques DORA.

Aucune intervention manuelle n'est nécessaire (hormis la création / validation de pull-request)

```mermaid

    flowchart TD

    %% =========================
    %% Déclenchement
    %% =========================

    A["push<br/>pull-request<br/>merge pull-request"]
    --> B["🔒 Sécurité<br/>pre-hook<br/>Détection secrets"]

    B --> C["Déclenchement<br/>GitHub Actions"]


    %% =========================
    %% PIPELINE CI
    %% =========================

    subgraph CI["Pipeline CI"]

        direction TB

        C --> FE["Build Front-End"]
        C --> BE["Build Back-End"]

        FE --> FES["🔒 Sécurité / Qualité<br/>SAST / CSA"]
        FE --> FET["Tests unitaires<br/>Front-End"]

        BE --> BET["Tests unitaires<br/>Back-End"]
        BE --> BES["🔒 Sécurité / Qualité<br/>SAST / CSA"]

        FES --> QG{"Quality<br/>Gates"}
        FET --> QG
        BET --> QG
        BES --> QG

        QG -->|❌ Échec| CIERR["Pipeline bloqué<br/>Erreur"]
        QG -->|✅ Succès| BRANCH{"Branche ?"}

        BRANCH -->|feature branch| STOP["Arrêt du pipeline<br/>Feedback"]
    end


    %% =========================
    %% PIPELINE RELEASE
    %% =========================

    BRANCH -->|develop / main| TAG

    subgraph RELEASE["Pipeline Release"]

        direction TB

        TAG["Calcul image tag<br/>semantic-release<br/>ex: v1.1.0"]

        TAG --> DOCKER["Docker build<br/>Back-End<br/>Front-End"]

        DOCKER --> CS["🔒 Sécurité<br/>Scan CS (Conteneur)"]

        CS -->|❌ Échec| RELERR["Pipeline bloqué<br/>Erreur"]

        CS -->|✅ Succès| PUSH["Push artefacts<br/>(GH Registry)"]

    end


    %% =========================
    %% PIPELINE CD
    %% =========================

    PUSH --> ENV{"Branche ?"}

    subgraph CD["Pipeline CD"]

        direction TB

        ENV -->|develop| INT["Déploiement<br/><b>INTEGRATION</b>"]

        INT --> DAST_INT["🔒 Sécurité<br/>Scan DAST"]

        DAST_INT -->|❌ Échec| CDERR["Pipeline bloqué<br/>Erreur"]


        ENV -->|main| PROD["Déploiement progressif<br/><b>PRODUCTION</b>"]

        PROD --> DAST_PROD["🔒 Sécurité<br/>Scan DAST"]

        DAST_PROD -->|❌ Échec| ROLLBACK["Rollback"]

        DAST_PROD -->|✅ Succès| FIN["SUITE / FIN<br/>Déploiement progressif"]

        FIN --> VERSION["vX.Y.Z ✅"]

    end

```