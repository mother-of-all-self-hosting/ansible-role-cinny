<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Linkwarden

This is an [Ansible](https://www.ansible.com/) role which installs [Linkwarden](https://linkwarden.app/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Linkwarden is a self-hosted, open-source collaborative bookmark manager to collect, organize and archive webpages.

See the project's [documentation](https://docs.linkwarden.app) to learn what Linkwarden does and why it might be useful to you.

## Prerequisites

To run a Linkwarden instance it is necessary to prepare a [Postgres](https://www.postgresql.org/) database server.

If you are looking for an Ansible role for Postgres, you can check out [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Linkwarden with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# linkwarden                                                           #
#                                                                      #
########################################################################

linkwarden_enabled: true

########################################################################
#                                                                      #
# /linkwarden                                                          #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Linkwarden you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
linkwarden_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Linkwarden under a subpath (by configuring the `linkwarden_path_prefix` variable) does not seem to be possible due to Linkwarden's technical limitations.

### Set variables for connecting to a Postgres database server

To have the Linkwarden instance connect to your Postgres server, add the following configuration to your `vars.yml` file.

```yaml
linkwarden_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
linkwarden_database_port: 5432
linkwarden_database_username: YOUR_POSTGRES_SERVER_USERNAME_HERE
linkwarden_database_password: YOUR_POSTGRES_SERVER_PASSWORD_HERE
linkwarden_database_name: YOUR_POSTGRES_SERVER_DATABASE_NAME_HERE
```

Make sure to replace values for variables with yours.

### Set a random string

You also need to set a random string used for session management. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `pwgen -s 64 1` or in another way.

```yaml
linkwarden_environment_variables_nextauth_secret: YOUR_SECRET_KEY_HERE
```

### Enabling signing up

By default this role is configured to disable signing up for an account on the service. To enable it, add the following configuration to your `vars.yml` file:

```yaml
linkwarden_environment_variables_next_public_disable_registration: false
```

### Connecting to a Meilisearch instance (optional)

To enable the [advanced search options](https://docs.linkwarden.app/Usage/advanced-search), you can optionally have the Linkwarden instance connect to a Meilisearch instance by adding the following configuration to your `vars.yml` file:

```yaml
linkwarden_environment_variables_meili_host: YOUR_MEILISEARCH_HOSTNAME_HERE
linkwarden_environment_variables_meili_key: YOUR_MEILISEARCH_KEY_HERE
```

>[!NOTE]
> The default Admin API Key is sufficient for using Meilisearch on a Linkwarden instance. It is [not recommended](https://www.meilisearch.com/docs/learn/security/basic_security) to use the master key for operations anything but managing other API keys.

If you are looking for an Ansible role for Meilisearch, you can check out [ansible-role-meilisearch](https://github.com/mother-of-all-self-hosting/ansible-role-meilisearch) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

### Configuring a SMTP mailer (optional)

You can configure a SMTP mailer to enable email functions such as password recovery and email address verification.

To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
linkwarden_environment_variables_next_public_email_provider: true

# Specify the email address from which will send the verification emails
linkwarden_environment_variables_email_from: ""

# Specify the url-encoded string with your credentials and the SMTP server
# Example: smtp://user:password@host:port
linkwarden_environment_variables_email_server: ""
```

See [this section](https://docs.linkwarden.app/self-hosting/environment-variables#smtp-settings) on the official documentation for details.

>[!NOTE]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `linkwarden_environment_variables_additional_variables` variable

See its [environment variables](https://raw.githubusercontent.com/linkwarden/linkwarden/refs/heads/main/.env.sample) for a complete list of Linkwarden's config options that you could put in `linkwarden_environment_variables_additional_variables`.

>[!NOTE]
> Not all environment variables are documented on [this page](https://docs.linkwarden.app/self-hosting/environment-variables).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Linkwarden becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and register the account. **Note that the first registered user becomes an administrator automatically.**

Since account registration is disabled by default, you need to enable it first by setting `linkwarden_environment_variables_next_public_disable_registration` to `false` temporarily in order to create your own account.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu linkwarden` (or how you/your playbook named the service, e.g. `mash-linkwarden`).
