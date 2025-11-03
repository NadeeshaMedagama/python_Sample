# choreo-control-plane

## Introduction

This repository contains the kustomize manifests (base) and Helm charts for deploying Choreo Control Plane (CP) and
Choreo Cloud Dataplane (CDP) services. The manifests are used to deploy the services in different environments. We are
using kustomize overlays in dev, stage and prod branches of choreo-cp-env-overlay repository to maintain environment
specific configurations for kustomize based deployments (Azure). The same branches contain the helm values files for
Helm based deployments (AWS). The kustomize manifests will soon be migrated to the helm charts as well.

- Kustomize manifests: [kustomize](kustomize)
- Helm charts: [cdp-helm-charts](cdp-helm-charts)

## Contributing

You need to send same change to both kustomize and helm charts. For helm syntax you can refer to
the [official Helm Documentation](https://helm.sh/docs/) (Or you can learn the syntax by going through the current helm
charts).

### Sending Changes to Helm Chart

Our two helm charts are designed as umbrella charts. Both charts follow the same structure and naming conventions.

#### Directory Structure

Each namespace has its own chart directory, which contains a `Chart.yaml` file and a `charts` directory that contains
the individual component charts. The directory structure is as follows:

```
├── charts
│   ├── choreo-ai <namespace>
│   │   ├── Chart.yaml
│   │   └── charts <components>
│   │       ├── copilot-datacollector
│   │       ├── cost-optimizer
│   │       ├── docs-updater
│   │       ├── milvus-proxy
│   │       └── spec-populator
│   ├── choreo-apim
│   │   ├── Chart.yaml
│   │   ├── charts
│   │   │   ├── apim-analytics
│   │   │   ├── apim-devportal
│   │   │   ├── app-gateway-oauth-agent
│   │   │   ├── choreo-connect
│   │   │   ├── choreo-connect-global-adapter
│   │   │   ├── choreo-governance-service
│   │   │   ├── choreo-linter-service
│   │   │   └── product-apim
```

You can use `helm create <chart-name>` command to create a new chart. This will create the basic structure for the
chart. You can then add/remove the necessary files and directories as per the above structure.

#### Helm Chart Naming and Resource Conventions

1. All kubernetes resource names should be as same as the component name. (There can be exceptions like CRDs)
2. Each kubernetes resource should have a label with the key `choreo.component` and the value as the component name. It
   is
   favourable to have only this label in the component resources unless there is a specific need for additional labels.
   These labels are defined in _helpers.tpl file of each component chart.
3. All new components should not expose secrets as environment variables. Instead, they should read the secret value
   from a file mounted to the pod.
4. With the point 3 in place, you can avoid the `.spec.secretObjects` section in the secret-provider-class.
5. The file name should match the kubernetes resource kind. (deployment.yaml, service.yaml, hpa.yaml,
   service-account.yaml, secret-provider-class.yaml, config.yaml etc.)
6. One component may have at most one SecretProviderClass and one ConfigMap.
7. All secret references should be specified in the values.yaml under `.secrets` node.
8. All config references should be specified in the values.yaml under `.configs` node.
9. Only use quotes for YAML values when required (e.g., values with special characters, reserved words, or ambiguous
   types). Avoid unnecessary quotes for simple strings.
10. The values.yaml in each sub chart contain sensible defaults for all the configurations. You can override
    these values in the overlay values.yaml files.
11. Think about the long term readability of the values.yaml;i.e: when it grows.(Ex: yaml array values are preferred
    over comma separated strings)

## Hotfixing Controlplane Repository

The following steps assumes that you want to hotfix a change in the choreo-control-plane repository to Staging
environment. (It is the overlay branch that changes based on the target environment)

1. Go to the stage-ci branch of choreo-cp-env-overlay repository and open the `kustomize/basev2.version` file.
2. Copy the content of the above file. This is the commit hash or tag of the choreo-control-plane repository that is
   currently deployed in the staging environment.
3. Go to the choreo-control-plane repository and checkout the above tag or commit hash.

```shell
#if it is a commit hash
git fetch upstream
git checkout -b hotfix <the copied commit hash>

#if it is a tag
git fetch upstream --tags
git checkout -b hotfix tags/<the copied tag> 
```

4. Do your changes and commit them to the hotfix branch.
5. Tag your changes in the following format.

```shell
git tag vhotfix-yyyymmdd-n # n is the hotfix number for the day
```

6. Push the tag to the upstream repository.

```shell
git push upstream vhotfix-yyyymmdd-n
```

7. Post the tag in the relevant release thread. Add the pull requests that are included in the hotfix to the
   release thread as a reference.
8. Make sure to immediately do the 7th step after pushing the tag, otherwise you will have to redo the process if
   another hotfix is applied before you do the 7th step (since the tag or commit hash you copied in the 2nd step changes
   per release/hotfix)

## Building an environment

In order to build an environment, the following prerequisites must be satisfied.

1. [choreo-control-plane](https://github.com/wso2-enterprise/choreo-control-plane)
   and [choreo-cp-env-overlay](https://github.com/wso2-enterprise/choreo-cp-env-overlay) repositories are cloned in the
   same directory level.
2. [main-ci](https://github.com/wso2-enterprise/choreo-control-plane/tree/main-ci) branch is checked out in
   choreo-control-plane repository.
    - If you want to use the [main](https://github.com/wso2-enterprise/choreo-control-plane/tree/main) branch instead,
      please remove the `-ci` from
      the https://github.com/wso2-enterprise/choreo-cp-env-overlay/blob/dev/kustomize/dev/kustomization.yaml#L5 or
      respective environment.
3. Related environment branch (dev, stage, or prod) is checked out
   in [choreo-cp-env-overlay](https://github.com/wso2-enterprise/choreo-cp-env-overlay).

Once you have met all the prerequisites, you can build the kustomize/env branch in choreo-cp-env-overlay folder.
Ex: kustomize build kustomize/dev

## Choreo Services Overview

### Control Plane (CP)

#### CP Services

| K8S Service Name (IaC Repository)                                                                                                                                                                  | Source Repository                                                                                                                                     | Description                                                            |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| [adapter](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-connect/charts/adapter)                                  | [Choreo Connect Global Adapter](https://github.com/wso2-enterprise/choreo-connect-global-adapter)                                                     | Global adapter for Choreo Connect                                      |
| [alert-notification-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/alert-notification-service)                | [Alert Notification Service](https://github.com/wso2-enterprise/choreo-alert-notification-service)                                                    | Handles alert notifications delivery                                   |
| [analytics-subscription](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/analytics-subscription)                        | [Analytics Subscriptions](https://github.com/wso2-enterprise/choreo-subscription-mgt)                                                                 | Manages analytics data subscriptions                                   |
| [api-key-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/api-key-service)                                      | [API Key Service](https://github.com/wso2-enterprise/choreo-iam/)                                                                                     | Handles API key generation and validation                              |
| [api-server](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/api-server)                                                | [API](https://github.com/wso2-enterprise/choreo-api)                                                                                                  | API definitions library containing proto files for all Choreo services |
| [apim-devportal](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/apim-devportal)                                          | [APIM DevPortal](https://docs.google.com/spreadsheets/d/10Gnpskv7oNACn28EhpbDSEx4QWFwOSe5K4RFGhK7Asc/edit?gid=50684561#gid=50684561)                  | Developer portal for API Manager                                       |
| [apim-proxy-deployer](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/apim-proxy-deployer)                              | [APIM Proxy Deployer](https://github.com/wso2-enterprise/choreo-apim-proxy-deployer/blob/main/choreo-deployment-guide.md)                             | Deploys API Manager proxies                                            |
| [app-dev-authz-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/app-dev-authz-service)                          | [AppDev Authorization](https://github.com/wso2-enterprise/choreo-appdev-authorization/)                                                               | Manages authorization for app development                              |
| [app-dev-user-mgt-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/app-dev-user-mgt-service)                    | [AppDev User Management](https://github.com/wso2-enterprise/choreo-appdev-user-mgt/blob/main/README.md#configuration)                                 | Handles user management for app development                            |
| [app-gateway-oauth-agent](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/app-gateway-oauth-agent)                        | [App Gateway OAuth Agent](https://github.com/wso2-enterprise/choreo-app-gateway-oauth-agent?tab=readme-ov-file#configurations)                        | OAuth agent for application gateway                                    |
| [app-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/app-service)                                              | [App Service](https://github.com/wso2-enterprise/choreo-app-service)                                                                                  | Manages application lifecycle                                          |
| [audit-logging](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/audit-logging)                                          | [Audit Logging](https://github.com/wso2-enterprise/choreo-audit-logging)                                                                              | Provides audit logging capabilities                                    |
| [cc-deployment](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-connect/charts/cc-deployment)                      | [Choreo Connect Deployment](https://github.com/wso2-enterprise/choreo-connect-deployment-control-plane-external-p1)                                   | Deploys Choreo Connect                                                 |
| [choreo-admin-api](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/choreo-admin-api)                                    | [Admin API](https://github.com/wso2-enterprise/choreo-admin-api)                                                                                      | Administrative API for Choreo platform                                 |
| [choreo-aws-marketplace-client](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/choreo-aws-marketplace-client)          | [AWS Marketplace Client](https://github.com/wso2-enterprise/choreo-aws-marketplace-client)                                                            | Integrates with AWS marketplace                                        |
| [choreo-connect-global-adapter](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-connect-global-adapter)            | [Connect Global Adapter](https://github.com/wso2-enterprise/choreo-connect-global-adapter/)                                                           | Global adapter for Choreo Connect                                      |
| [choreo-console](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/choreo-console)                                        | [Console](https://github.com/wso2-enterprise/choreo-console)                                                                                          | Choreo console UI                                                      |
| [choreo-gcp-marketplace-client](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/choreo-gcp-marketplace-client)          | [GCP Marketplace Client](https://github.com/wso2-enterprise/choreo-gcp-marketplace-client)                                                            | Integrates with GCP marketplace                                        |
| [choreo-governance-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-governance-service)                    | [Governance Service](https://github.com/wso2-enterprise/choreo-governance-service)                                                                    | Provides governance features                                           |
| [choreo-linter-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-linter-service)                            | [Linter Service](https://github.com/wso2-enterprise/choreo-linter-service)                                                                            | Provides code linting capabilities                                     |
| [choreo-url-management](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/choreo-url-management)                          | [URL Management](https://github.com/wso2-enterprise/choreo-url-management)                                                                            | Manages URL configurations                                             |
| [cio-event-collector-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cio-event-collector-service)              | [CIO Event Collector Service](https://github.com/wso2-enterprise/choreo-cio-dashboard/blob/main/README.md)                                            | Collects operational events                                            |
| [cio-incident-configurator](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cio-incident-configurator)                  | [CIO Incident Configurator](https://github.com/wso2-enterprise/choreo-cio-dashboard/blob/main/README.md)                                              | Configures incident management                                         |
| [cio-query-api-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cio-query-api-service)                          | [CIO Query API Service](https://github.com/wso2-enterprise/choreo-cio-dashboard/blob/main/README.md)                                                  | Provides query API for operational data                                |
| [config-deployer-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/config-deployer-service)                        | [Config Deployer Service](https://github.com/wso2-enterprise/config-deployer-service)                                                                 | Deploys configurations                                                 |
| [configuration-schema-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/configuration-schema-service)            | [Configuration Schema Service](https://github.com/wso2-enterprise/configuration-schema-service)                                                       | Manages configuration schemas                                          |
| [configuration-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/configuration-service)                          | [Configuration Service](https://github.com/wso2-enterprise/choreo-cp-configuration-service?tab=readme-ov-file#configuration-service--mapping-service) | Handles platform configurations                                        |
| [connection-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/connection-service)                                | [Connection Service](https://github.com/wso2-enterprise/choreo-marketplace?tab=readme-ov-file#connection-service)                                     | Manages external connections                                           |
| [contract-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/contract-service)                                    | [Contract Service](https://github.com/wso2-enterprise/choreo-contract-service)                                                                        | Manages service contracts                                              |
| [cp-graphql](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cp-graphql)                                                | [CP GraphQL](https://github.com/wso2-enterprise/choreo-cp-graphql?tab=readme-ov-file#setting-up-the-graphql-server-in-a-cluster)                      | GraphQL interface for control plane                                    |
| [cp-gw-adapter](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cp-gw-adapter)                                          | [CP Gateway Adapter](https://github.com/wso2-enterprise/choreo-cp-gateway-adapter)                                                                    | Adapter for control plane gateway                                      |
| [cp-resilient-invoker](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cp-resilient-invoker)                            | [CP Resilient Invoker](https://github.com/wso2-enterprise/choreo-cp-resilient-invoker)                                                                | Provides resilient service invocation                                  |
| [cp-rrs](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cp-rrs)                                                        | [CP RRS](https://github.com/wso2-enterprise/choreo-cp-rrs)                                                                                            | Remote runtime service                                                 |
| [copilot-datacollector-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/copilot-datacollector-service)          | [Copilot Data Collector Service](https://github.com/wso2-enterprise/choreo-ai-copilot/blob/main/README.md)                                            | Collects data for AI copilot                                           |
| [cost-optimizer-cp-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cost-optimizer-cp-service)                  | [Cost Optimizer CP Service](https://github.com/wso2-enterprise/choreo-billing/blob/main/cost-optimizer-cp-service/README.md)                          | Optimizes costs for control plane                                      |
| [cp-daily-cron](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/cp-daily-cron)                                          | [CP Daily Cron](https://github.com/wso2-enterprise/choreo-billing/blob/main/cost-optimizer-cronjobs/cp-daily-cron/README.md)                          | Daily scheduled tasks                                                  |
| [crypto-key-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/crypto-key-service)                                | [Crypto Key Service](https://github.com/wso2-enterprise/choreo-crypto-key-service)                                                                    | Manages cryptographic keys                                             |
| [declarative-api](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/declarative-api)                                      | [CP Declarative API](https://github.com/wso2-enterprise/choreo-cp-declarative-api?tab=readme-ov-file#choreo-cp-declarative-api)                       | Provides declarative API capabilities                                  |
| [delete-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/delete-manager)                                        | [Delete Manager](https://github.com/wso2-enterprise/choreo-delete-manager?tab=readme-ov-file#deployment-configs-and-secrets)                          | Manages resource deletion                                              |
| [devops-portal-api](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/devops-portal-api)                                  | [DevOps Portal API](https://github.com/wso2-enterprise/choreo-devops-portal-api)                                                                      | API for DevOps portal                                                  |
| [docs-updater](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/docs-updater)                                            | [Docs Updater](https://github.com/wso2-enterprise/ai-docs-assistant/blob/main/README.md)                                                              | Updates documentation                                                  |
| [dp-cicd](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-cicd)                                                      | [DP CICD](https://github.com/wso2-enterprise/choreodp-cicd)                                                                                           | CI/CD for data plane                                                   |
| [dp-cloud-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-cloud-manager)                                    | [DP Cloud Manager](https://github.com/wso2-enterprise/choreodp-cloud-manager?tab=readme-ov-file#deployment-configs-and-secrets)                       | Manages cloud resources for data plane                                 |
| [dp-garbage-collector](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-garbage-collector)                            | [DP Garbage Collector](https://github.com/wso2-enterprise/dp-garbage-collector)                                                                       | Cleans up data plane resources                                         |
| [dp-mizzen](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-mizzen)                                                  | [DP Mizzen](https://github.com/wso2-enterprise/choreodp-mizzen?tab=readme-ov-file#deployment-configuration)                                           | Data plane component                                                   |
| [dp-project-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-project-manager)                                | [DP Project Manager](https://github.com/wso2-enterprise/choreodp-project-manager?tab=readme-ov-file#setting-up-the-project-manager-in-a-cluster)      | Manages data plane projects                                            |
| [dp-rudder](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-rudder)                                                  | [DP Rudder](https://github.com/wso2-enterprise/choreodp-rudder)                                                                                       | Data plane steering component                                          |
| [dp-secret-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/dp-secret-manager)                                  | [DP Secret Manager](https://github.com/wso2-enterprise/choreodp-secret-manager?tab=readme-ov-file#deployment-configuration)                           | Manages secrets for data plane                                         |
| [email-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/email-service)                                          | [Email](https://github.com/wso2-enterprise/choreo-email)                                                                                              | Handles email communications                                           |
| [endpoint-resolver](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/endpoint-resolver)                                  | [Endpoint Resolver](https://github.com/wso2-enterprise/choreo-marketplace?tab=readme-ov-file#endpoint-resolver)                                       | Resolves service endpoints                                             |
| [global-adapter-health-check-client](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/global-adapter-health-check-client)  | [GA Health Check Client](https://github.com/wso2-enterprise/choreo-ga-health-check-client)                                                            | Health check client for global adapter                                 |
| [log-controller](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/log-controller)                                        | [Log Controller](https://github.com/wso2-enterprise/choreo-log-controller)                                                                            | Controls logging operations                                            |
| [marketplace](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/marketplace)                                              | [Marketplace](https://github.com/wso2-enterprise/choreo-marketplace?tab=readme-ov-file#endpoint-resolver)                                             | Manages marketplace offerings                                          |
| [milvus-proxy-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/milvus-proxy-service)                            | [Milvus Proxy Service](https://github.com/wso2-enterprise/apim-ai-deployments/blob/main/README.md)                                                    | Proxy for Milvus vector database                                       |
| [moesif-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/moesif-service)                                        | [Moesif Microservice](https://github.com/wso2-enterprise/choreo-moesif-microservice)                                                                  | Integration with Moesif API analytics                                  |
| [obsmanager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/obsmanager)                                                | [Obs Manager](https://github.com/wso2-enterprise/choreo-obs-manager/blob/main/README.md)                                                              | Manages observability features                                         |
| [organization-management](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/organization-management)                      | [Organization Management](https://github.com/wso2-enterprise/choreo-organization-management)                                                          | Manages organizations                                                  |
| [organization-remover](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/organization-remover)                            | [Organization Remover](https://github.com/wso2-enterprise/choreo-organization-remover)                                                                | Handles organization removal                                           |
| [pdp-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/pdp-manager)                                              | [PDP Manager](https://github.com/wso2-enterprise/choreo-pdp-manager)                                                                                  | Manages policy decision points                                         |
| [platform-services-manager](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/platform-services-manager)                  | [Platform Services Manager](https://github.com/wso2-enterprise/choreo-platform-services-manager)                                                      | Manages platform services                                              |
| [product-apim](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/product-apim)                                              | [Product APIM](https://github.com/malinthaprasan/choreo-product-apim/blob/pr-template/README.md)                                                      | Product packaging for API Manager                                      |
| [rate-limiter](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-apim/charts/choreo-connect/charts/rate-limiter)                        | [Quota Limiter](https://github.com/wso2-enterprise/choreo-quota-limiter)                                                                              | Enforces usage quotas                                                  |
| [resource-authorization-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-authorization/charts/resource-authorization-service) | [Resource Authorization Service](https://github.com/wso2-enterprise/choreo-resource-authorization-service/blob/main/README.md)                        | Authorizes resource access                                             |
| [resource-registry](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/resource-registry)                                  | [Resource Registry](https://github.com/wso2-enterprise/choreo-marketplace?tab=readme-ov-file#resource-registry-service)                               | Registers and tracks resources                                         |
| [spec-populator-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/spec-populator-service)                        | [Spec Populator Service](https://github.com/wso2-enterprise/apim-ai-deployments/blob/main/README.md)                                                  | Populates API specifications                                           |
| [sts-mgt-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/sts-mgt-service)                                      | [STS Management Service](https://github.com/wso2-enterprise/choreo-iam/)                                                                              | Manages security token service                                         |
| [subscriptions-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/subscriptions-service)                          | [Subscriptions](https://github.com/wso2-enterprise/choreo-subscriptions)                                                                              | Handles service subscriptions                                          |
| [usagehandler-service](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/usagehandler-service)                            | [Metered Usage Handler](https://github.com/wso2-enterprise/choreo-metered-usage-handler)                                                              | Handles metered usage data                                             |
| [workflow-mgt](https://github.com/wso2-enterprise/choreo-control-plane/tree/main/cdp-helm-charts/controlplane/charts/choreo-system/charts/workflow-mgt)                                            | [Workflow Management](https://github.com/wso2-enterprise/choreo-workflow-mgt/)                                                                        | Manages workflows                                                      |
