# Security policy

## Reporting a vulnerability

**Do not open a public issue.** Email **hello@kanunilabs.com** with
`SECURITY` in the subject line, and include:

- what the issue is and how to trigger it;
- which package and version;
- what an attacker gains.

You will get an acknowledgement within **72 hours**. We will tell you what we
found, what we plan to do, and when — and we will credit you in the release
notes if you want that.

Please give us a reasonable window to ship a fix before disclosing publicly.
We are a small team, not a silent one: if something is going slowly you will
hear why rather than nothing.

## Scope

In scope: the published packages listed in the [README](README.md), the licence
verification mechanism, and <https://www.kanunilabs.com>.

Out of scope, because they are already documented as designed:

- **The licence key is not a secret.** It is a signed, non-encrypted token that
  ships in your client bundle and is meant to be readable there. It carries no
  credentials, and reading one grants nothing beyond what the customer already
  bought. See
  [why a licence key is not a secret](https://www.kanunilabs.com/blog/why-a-licence-key-is-not-a-secret).
- Automated scanner output with no demonstrated impact.
- Missing hardening headers on pages that carry no session or user data.

## What is genuinely interesting

Anything that lets someone **use Enterprise features without a valid licence**,
**read another customer's data**, **act on another customer's account**, or
**obtain private-registry credentials**. Those we want to hear about
immediately.
