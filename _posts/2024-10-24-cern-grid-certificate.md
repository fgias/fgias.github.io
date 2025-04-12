---
title: "Obtaining and Renewing a CERN Grid Certificate in LHCb"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - LHCb
  - CERN
---

## Obtaining a Certificate

Instructions [here](https://twiki.cern.ch/twiki/bin/view/LHCb/FAQ/Certificate).

- Always use Firefox

- Import all the certificates on the browser

- Import you own certificate to the browser, and keep the import password

- Send the file to your lxplus account with scp

- Log in: 

```
ssh -Y username@lxplus.cern.ch
```

- Convert to PEM and use a PEM passphrase different from the import password: 

```
lb-dirac dirac-cert-convert MyFile.p12
```

- Join the LHCb Virtual Organisation, and wait 24 hours for validation, see [link](https://twiki.cern.ch/twiki/bin/view/LHCb/FAQ/Certificate)

- After validation, in lxplus try: `lhcb-proxy-init`, and enter the PEM passphrase

- Now you can access the LHCb DIRAC portal

## Renewing a Certificate

- Download a certificate from the [Certification Authority](https://ca.cern.ch/ca/).

- Move it to lxplus with `scp`.

- Delete the old ceritificate from the Firefox browser.

- On [IAM](https://lhcb-auth.cern.ch/), on your profile, under `X.509 certificates`, unlink the old certificate.

- Import the `.p12` certificate to Firefox.

- On the same IAM page, link the new certificate to your account.

- On lxplus, delete everything in the `~/.globus` directory.

- Like before, convert to PEM and use a PEM passphrase different from the import password: 

```
lb-dirac dirac-cert-convert MyFile.p12
```
- Check that everything works as it should [here](https://cern.service-now.com/service-portal?id=kb_article&n=KB0003774).

- Possibly run `lhcb-dirac`.

- Finally, run `lhcb-proxy-init`.
