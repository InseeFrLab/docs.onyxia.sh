## Community Call 2026/06/26

### Work on S3 profiles : 
- add of a feature for chrome navigator : stream possibilities instead of loading files in memory
- wip : inject the different s3 profiles in interactive services through `.aws/config` configuration files.

### Plugin System
In the frontpage, there is now the possibility to add a card so that users can launch a specific service with a step by step guidance to add gitlab configuration (such as the project to clone)

### Marimo: new interactive service 
- [Marimo](https://marimo.io/) docker image was added to the list of images we build and the corresponding helm chart should be added before next community call. To follow the progress :  https://github.com/InseeFrLab/helm-charts-interactive-services/pull/313

### Opencode was added to our ide images
When launching an interactive service, the configuration is automatically injected if the correct configuration was filled in within the user profile. See: https://github.com/InseeFrLab/images-datascience/pull/378
