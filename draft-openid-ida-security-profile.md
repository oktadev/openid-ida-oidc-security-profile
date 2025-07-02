---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Identity Assurance Profile for OpenID Connect"
abbrev: "IDA Profile for OIDC"
category: info

docname: draft-openid-ida-security-profile-latest
submissiontype: "independent"
number:
date:
v: 3
#area: AREA
workgroup: eKYC-IDA
keyword:
 - identity assurance
venue:
  group: eKYC-IDA
  type: Working Group
  mail: openid-specs-ekyc-ida@lists.openid.net
  arch: https://openid.net/wg/ekyc-ida/
  github: oktadev/openid-ida-oidc-security-profile
  latest: https://github.com/oktadev/openid-ida-oidc-security-profile

author:
 -
    fullname: Scott Bermon
    organization: Okta
    email: scott.bermon@okta.com
 -
    fullname: Aaron Parecki
    organization: Okta
    url: https://aaronparecki.com

normative:

  OpenID:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore

informative:



--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
