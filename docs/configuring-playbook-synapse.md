# Configuring Synapse (optional)

By default, this playbook configures the [Synapse](https://github.com/matrix-org/synapse) Matrix server, so that it works for the general case.
If that's enough for you, you can skip this document.

The playbook provides lots of customization variables you could use to change Synapse's settings.

Their defaults are defined in [`roles/matrix-synapse/defaults/main.yml`](../roles/matrix-synapse/defaults/main.yml) and they ultimately end up in the generated `/matrix/synapse/config/homeserver.yaml` file (on the server). This file is generated from the [`roles/matrix-synapse/templates/synapse/homeserver.yaml.j2`](../roles/matrix-synapse/templates/synapse/homeserver.yaml.j2) template.

**If there's an existing variable** which controls a setting you wish to change, you can simply define that variable in your configuration file (`inventory/host_vars/matrix.<your-domain>/vars.yml`) and [re-run the playbook](installing.md) to apply the changes.

Alternatively, **if there is no pre-defined variable** for a Synapse setting you wish to change:

- you can either **request a variable to be created** (or you can submit such a contribution yourself). Keep in mind that it's **probably not a good idea** to create variables for each one of Synapse's various settings that rarely get used.

- or, you can **extend and override the default configuration** ([`homeserver.yaml.j2`](../roles/matrix-synapse/templates/synapse/homeserver.yaml.j2)) by making use of the `matrix_synapse_configuration_extension_yaml` variable. You can find information about this in [`roles/matrix-synapse/defaults/main.yml`](../roles/matrix-synapse/defaults/main.yml).

- or, if extending the configuration is still not powerful enough for your needs, you can **override the configuration completely** using `matrix_synapse_configuration` (or `matrix_synapse_configuration_yaml`). You can find information about this in [`roles/matrix-synapse/defaults/main.yml`](../roles/matrix-synapse/defaults/main.yml).

## Synapse OpenID Connect

If you plan to rely only on SSO and don't want your users to ever login with password credentials disable this:
```yaml
matrix_synapse_password_config_enabled: true
```

Add this to allow seamless forwarding to element web app and element android app. Without this setting matrix will ask the user if he trusts the app he tries to login.  

```yaml
matrix_synapse_sso_client_whitelist:
  - "https://element.{{ matrix_domain }}/"
  - element://element

```

If you use `nginx proxy role` then you should add this setting as well to successfully pass redirects: 
```yaml
matrix_nginx_proxy_proxy_matrix_client_api_forwarded_location_synapse_oidc_api_enabled: true

```

Refer to [official Synapse doc](https://github.com/matrix-org/synapse/blob/develop/docs/openid.md) for oidc setup examples.


This is the bare minimum config example:
```yaml
# Configure open id connect
matrix_synapse_oidc_enabled: true
matrix_synapse_oidc_issuer: "https://openid.example.com"
matrix_synapse_oidc_client_id: "your-client-id"
matrix_synapse_oidc_client_secret: "s0m3v3RyS3cr3tStr!ng"

```

Depending on oauth provider you may also like to change following settings:

```yaml
matrix_synapse_oidc_scopes: ["openid"]
matrix_synapse_oidc_allow_existing_users: false
matrix_synapse_oidc_client_auth_method: client_secret_basic
matrix_synapse_oidc_subject_claim: "sub"
matrix_synapse_oidc_localpart_template: "{% raw %}{{ user.preferred_username }}{% endraw %}"
matrix_synapse_oidc_display_name_template: "{% raw %}{{ user.name }}{% endraw %}"
```

## Synapse Admin

Certain Synapse administration tasks (managing users and rooms, etc.) can be performed via a web user-interace, if you install [Synapse Admin](configuring-playbook-synapse-admin.md).


## Synapse + OpenID Connect for Single-Sign-On

If you'd like to use OpenID Connect authentication with Synapse, you'll need some additional reverse-proxy configuration (see [our nginx reverse-proxy doc page](configuring-playbook-nginx.md#synapse-openid-connect-for-single-sign-on)).
