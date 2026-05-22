# January 2026 community call

### January 2026 community call

#### Welcome back Marc

* Marc, our primary designer (2020-2022) came back home last month

#### Onyxia communication

* https://community.ima-dt.org/france-corporate-innovation-award-2026/content/liste-nomines we won the european sovereignty category.
* Cloud native Day Paris february 3 : we have a booth, come to talk to us :)

#### Onyxia news

* still working on file explorer (supporting profile file semantic, bookmark, enhance UX)
* support of postgres CNPG
* Planned support of Iceberg Rest API. Onyxia will be able to use this king of lakehouse as it perfectly fit cloud native env.
* Ingress-nginx project will be retired in March 2026 (soon !). If you are currently using it (SSPCloud is currently using it), what is your plan ? Onyxia has no major dependency to ingress-nginx but admins may want to take this deadline as an opportunity to migrate to gateway API. Onyxia support for gateway API is planned / in-progress but will probably not be ready by the march 2026 deadline (work is needed on onyxia-api but most importantly on charts). Context : https://kubernetes.io/blog/2026/01/29/ingress-nginx-statement/
* Seaweedfs (https://github.com/seaweedfs/seaweedfs) now supports policy variables (https://github.com/seaweedfs/seaweedfs/issues/8037) allowing to define policies such as `user johndoe has access to bucket user-johndoe` just like in minIO / AWS. You may find this project as a good replacment for minIO following their licensing changes
* DPOP ! onyxia is an SPA, we need to protect our access tokens. oidc-spa support DPOP : https://oauth.net/2/dpop/
