---
description: The frontend
---

# 🐲 onyxia-web

This is the documentation for [InseeFrLab/onyxia-web](https://github.com/InseeFrLab/onyxia-web). &#x20;

```bash
git clone https://github.com/InseeFrLab/onyxia-web
cd onyxia-web
git submodule update --init --recursive

# Download the binary files (images, fonts, ect, you need git LFS)
git lfs install && git lfs pull
yarn install
#Setup the var envs to tell the app to connect to the sspcloud
#Fill up with your own value to run the web app against your own infra.
cp .env.local.sample-insee .env.local

# To stat the app locally
yarn start 
```
