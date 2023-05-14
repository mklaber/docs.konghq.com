---
name: MK Auth
publisher: Klabetron Ltd

categories:
  - authentication
type: plugin
desc: Add API key authentication using an external provider
description: |
  Add API key authentication to your routes, services, or workspaces. Consumers specify their key in a request header. The key
  is then authenticated using an external provider that exchanges the key for a JWT. Claims from the returned JWT can be
  proxied to upstream services in custom headers.

  > The advantage of this plugin over the [Key Authentication plugin](/hub/kong-inc/key-auth/) is that it does not require the
  > use or configuration of Kong API Consumers. Instead, authentication is handled by an external provider.

support_url: https://github.com/mklaber/kong-plugin-auth/issues

source_code: https://github.com/mklaber/kong-plugin-auth

license_type: MIT

kong_version_compatibility:
  community_edition:
    compatible: true
  enterprise_edition:
    compatible: true

cloud: false
enterprise: false
plus: false

#########################
# PLUGIN-ONLY SETTINGS below this line

params:
  name: mkauth
  service_id: true
  consumer_id: false
  route_id: true
  protocols:
    - name: http
  dbless_compatible: yes
  config:
    - name: auth_server_url
      required: yes
      datatype: url
      value_in_examples: <auth-server-url>
      description: |
        The authentication server endpoint that accepts `GET` requests, that authenticates using the
        `auth_header_name` header, and returns a JWT.
    - name: auth_header_name
      required: no
      default: authentication
      datatype: string
      value_in_examples: authentication
      description: |
        The header from the incoming request that contains an API key.

        > This header should be the same name as the `auth_server_url` expects. If the header names need
        > to differ, consider using the [Request Transformer plugin](/hub/kong-inc/request-transformer/) to
        > rename the headers.
    - name: upstream_header_claims
      required: semi
      datatype: array of string elements
      value_in_examples: sub,aud
      description: |
        The names of claims to extract from the authentication server's JWT response. The extracted
        values will be sent upstream in `upstream_header_names`. If the JWT does not have one of these claims,
        its corresponding header will not be sent upstream.

        > This setting is required when `upstream_header_names` is specified and must be of the same length.
    - name: upstream_header_names
      required: semi
      datatype: array of string elements
      value_in_examples: x-mk-sub,x-mk-aud
      description: |
        The names of headers to send extracted claim values to, upstream. The claims to extract are defined in
        `upstream_header_claims`.

        > This setting is required when `upstream_header_claims` is specified and must be of the same length.

 
###############################################################################
# END YAML DATA

###############################################################################
# BEGIN MARKDOWN CONTENT
---

## Usage

The `mkauth` plugin makes it easy to combine Key Auth with JWT upstream proxying without requiring the use of Kong's consumer objects. This can make migrating existing infrastructure into Kong easier by supporting a legacy authentication provider.

The authentication provider, specified as `auth_server_url`, should, upon successful authentication, return a JWT token representing the identity of the API key.

1. Consumer authenticates a request: `curl -X POST -H Authentication:<api-key> http://mykonggateway/my/service`
2. The plugin calls `curl -X GET -H Authentication:<api-key> <auth-server-url>`
3. Optionally, the plugin extracts claims (`upstream_header_claims`) from the authentication provider's response and proxies them upstream (`upstream_header_names`)

The authentication provider must use header-based authentication using the same header as the plugin is configured to use (in `auth_header_name`). This means that the consumer must specify the key in the same header as the authentication provider expects. If you need these headers to be different, consider using the [Request Transformer plugin](/hub/kong-inc/request-transformer/) to rename headers.

## Installation

The plugin can be installed using `luarocks` (assuming you have access to the repository):

```bash
luarocks install https://raw.githubusercontent.com/mklaber/kong-plugin-auth/master/kong-plugin-mkauth-0.1.0-1.rockspec
```

## Changelog

**{{site.base_gateway}} 3.2.x**
* Initial release
