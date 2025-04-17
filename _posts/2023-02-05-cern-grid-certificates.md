---
title: "Obtaining and Renewing a CERN Grid Certificate in LHCb"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - LHCb
  - CERN
---

Working with LHCb resources requires a valid grid certificate. Here's a step-by-step guide for obtaining and renewing your certificate, tailored for lxplus and LHCb DIRAC access.

## Obtaining a Certificate

Instructions: [Official LHCb Certificate Guide](https://twiki.cern.ch/twiki/bin/view/LHCb/FAQ/Certificate)

1. **Use Firefox** — Chrome and other browsers may not work well for certificate management.

2. **Import all required certificates** into Firefox (as described in the link above).

3. **Import your personal certificate** into Firefox. You’ll be prompted to set an **import password** — remember it!

4. **Transfer the certificate** to your `lxplus` account:

   ```bash
   scp MyCertificate.p12 username@lxplus.cern.ch:
   ```

5. **SSH into lxplus**:

   ```bash
   ssh -Y username@lxplus.cern.ch
   ```

6. **Convert the certificate to PEM format** (you’ll be asked to set a **PEM passphrase** — use a different one than the import password):

   ```bash
   lb-dirac dirac-cert-convert MyCertificate.p12
   ```

7. **Join the LHCb Virtual Organisation (VO)** via the [VOMS registration page](https://twiki.cern.ch/twiki/bin/view/LHCb/FAQ/Certificate). You may need to wait ~24 hours for validation.

8. Once validated, initialize your proxy on `lxplus`:

   ```bash
   lhcb-proxy-init
   ```

   Enter your PEM passphrase when prompted.

✅ You now have access to the [LHCb DIRAC portal](https://lhcb-portal-dirac.cern.ch/).

## Renewing a Certificate

1. **Download your renewed certificate** from the [CERN Certification Authority](https://ca.cern.ch/ca/).

2. **Transfer it to lxplus**:

   ```bash
   scp NewCertificate.p12 username@lxplus.cern.ch:
   ```

3. **In Firefox**, delete the old certificate and import the new `.p12` one.

4. **Update your IAM profile**:
   - Visit [IAM Profile](https://lhcb-auth.cern.ch/)
   - Under `X.509 certificates`, **unlink the old certificate**.
   - Then, **link the new certificate** you just imported.

5. **On lxplus**, clear your previous credentials:

   ```bash
   rm -rf ~/.globus
   ```

6. **Convert the new certificate to PEM** as before:

   ```bash
   lb-dirac dirac-cert-convert NewCertificate.p12
   ```

7. **Test your setup**: Follow [this checklist](https://cern.service-now.com/service-portal?id=kb_article&n=KB0003774) to ensure everything is working.

8. Optionally run:

   ```bash
   lhcb-dirac
   ```

9. Finally, reinitialize your proxy:

   ```bash
   lhcb-proxy-init
   ```

✅ You have renewed your certificate.
