# February 2025 community call

Community call 02/28/2025

News :

* Rewrite of the onboarding as a go module : Going well, first version has been released yesterday : [https://github.com/onyxia-datalab/onyxia-onboarding](https://github.com/onyxia-datalab/onyxia-onboarding) . Current work is on the Helm chart with a standalone version already published and integration of this module as a optional dependency in the main Onyxia chart should arrive soon. Slack channel for discussion on the rewrite effort : #dev-rewrite-to-go
* Onyxia-web : Work on improving support for OIDC providers other than Keycloak. In particular Entra ID (Microsoft) and Auth0. Thoughough documentation will be added to the docs.onyxia.dev website shortly. [https://docs.oidc-spa.dev/](https://docs.oidc-spa.dev/)

Community contributions :

* Trygve : Great work on the new Go-based onboarding API so far! We just need to make sure the test coverage for the go rewrite is maintained at a good level to avoid reproducing the same mistakes as the current Java Onyxia-API (which has a ridiculous low level of test)
* Trygve : they developped a custom web plugin to display the estimated cost of running the service based on resources (cpu / mem) beside the resources slider. Really interesting but may be hard to opensource properly due to variety in setups and Charts
* Discussion on how to get billing usage : prometheus, opencost. Also hard to opensource / bundle to Onyxia / make it generic\
  ![image](https://hackmd.io/_uploads/Syp-qVki1g.png)
* Trygve : 200 users daily :heart:
* NTTS (eurostat conference) 11-13 march 2025. Come and say hi :relaxed:
* CSTB ([https://www.cstb.fr/](https://www.cstb.fr/)) : approx 1000 employees including datascientists. Currently running Jupyterhub and other tools. Interested in Onyxia for unifying tools among all projects and embrace opensource ecosystems. Welcome :relaxed:
