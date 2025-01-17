---
title: "Redshift setup"
description: "Read this guide to learn about the Redshift warehouse setup in dbt."
id: "redshift-setup"
meta:
  maintained_by: dbt Labs
  authors: 'core dbt maintainers'
  github_repo: 'dbt-labs/dbt-redshift'
  pypi_package: 'dbt-redshift'
  min_core_version: 'v0.10.0'
  cloud_support: Supported
  min_supported_version: 'n/a'
  slack_channel_name: '#db-redshift'
  slack_channel_link: 'https://getdbt.slack.com/archives/C01DRQ178LQ'
  platform_name: 'Redshift'
  config_page: '/reference/resource-configs/redshift-configs'
---

<Snippet src="warehouse-setups-cloud-callout" />

<h2> Overview of {frontMatter.meta.pypi_package} </h2>

<ul>
    <li><strong>Maintained by</strong>: {frontMatter.meta.maintained_by}</li>
    <li><strong>Authors</strong>: {frontMatter.meta.authors}</li>
    <li><strong>GitHub repo</strong>: <a href={`https://github.com/${frontMatter.meta.github_repo}`}>{frontMatter.meta.github_repo}</a><a href={`https://github.com/${frontMatter.meta.github_repo}`}><img src={`https://img.shields.io/github/stars/${frontMatter.meta.github_repo}?style=for-the-badge`}/></a></li>
    <li><strong>PyPI package</strong>: <code>{frontMatter.meta.pypi_package}</code> <a href={`https://badge.fury.io/py/${frontMatter.meta.pypi_package}`}><img src={`https://badge.fury.io/py/${frontMatter.meta.pypi_package}.svg`}/></a></li>
    <li><strong>Slack channel</strong>: <a href={frontMatter.meta.slack_channel_link}>{frontMatter.meta.slack_channel_name}</a></li>
    <li><strong>Supported dbt Core version</strong>: {frontMatter.meta.min_core_version} and newer</li>
    <li><strong>dbt Cloud support</strong>: {frontMatter.meta.cloud_support}</li>
    <li><strong>Minimum data platform version</strong>: {frontMatter.meta.min_supported_version}</li>
    </ul>


<h2> Installing {frontMatter.meta.pypi_package} </h2>

pip is the easiest way to install the adapter:

<code>pip install {frontMatter.meta.pypi_package}</code>

<p>Installing <code>{frontMatter.meta.pypi_package}</code> will also install <code>dbt-core</code> and any other dependencies.</p>

<h2> Configuring {frontMatter.meta.pypi_package} </h2>

<p>For {frontMatter.meta.platform_name}-specific configuration, refer to <a href={frontMatter.meta.config_page}>{frontMatter.meta.platform_name} Configuration</a>. </p>

<p>For further info, refer to the GitHub repository: <a href={`https://github.com/${frontMatter.meta.github_repo}`}>{frontMatter.meta.github_repo}</a>.</p>


## Authentication Methods

### Password-based authentication

<File name='~/.dbt/profiles.yml'>

```yaml
company-name:
  target: dev
  outputs:
    dev:
      type: redshift
      host: hostname.region.redshift.amazonaws.com
      user: username
      password: password1
      port: 5439
      dbname: analytics
      schema: analytics
      threads: 4
      connect_timeout: None # optional, number of seconds before connection times out 
      # search_path: public # optional, not recommended
      sslmode: prefer # optional, set the sslmode to connect to the database. Default prefer, which will use 'verify-ca' to connect.
      role: # optional
      ra3_node: true # enables cross-database sources
      autocommit: true # enables autocommit after each statement
      region: # optional, if not provided, will be determined from host (e.g. host.123.us-east-1.redshift-serverless.amazonaws.com)
```

</File>

### IAM Authentication

To set up a Redshift profile using IAM Authentication, set the `method`
parameter to `iam` as shown below. Note that a password is not required when
using IAM Authentication. For more information on this type of authentication,
consult the [Redshift Documentation](https://docs.aws.amazon.com/redshift/latest/mgmt/generating-user-credentials.html)
and [boto3
docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/redshift.html#Redshift.Client.get_cluster_credentials)
on generating user credentials with IAM Auth.

If you receive the "You must specify a region" error when using IAM
Authentication, then your aws credentials are likely misconfigured. Try running
`aws configure` to set up AWS access keys, and pick a default region. If you have any questions,
please refer to the official AWS documentation on [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).

<File name='~/.dbt/profiles.yml'>

```yaml
my-redshift-db:
  target: dev
  outputs:
    dev:
      type: redshift
      method: iam
      cluster_id: CLUSTER_ID
      host: hostname.region.redshift.amazonaws.com
      user: alice
      iam_profile: data_engineer # optional
      autocreate: true           # optional
      db_groups: ['ANALYSTS']    # optional

      # Other Redshift configs:
      port: 5439
      dbname: analytics
      schema: analytics
      threads: 4
      connect_timeout: None # optional, number of seconds before connection times out 
      [retries](#retries): 1 # default 1 retry on error/timeout when opening connections
      role: # optional
      sslmode: prefer # optional, set the sslmode to connect to the database. Default prefer, which will use 'verify-ca' to connect.
      ra3_node: true # enables cross-database sources
      autocommit: true # optional, enables autocommit after each statement
      region: # optional, if not provided, will be determined from host (e.g. host.123.us-east-1.redshift-serverless.amazonaws.com)


```

</File>

### Specifying an IAM Profile

When the `iam_profile` configuration is set, dbt will use the specified profile from your `~/.aws/config` file instead of using the profile name `default`

## Redshift notes

### `sslmode` change
Before to dbt-redshift 1.5, `psycopg2` was used as the driver. `psycopg2` accepts `disable`, `prefer`, `allow`, `require`, `verify-ca`, `verify-full` as valid inputs of `sslmode`, and does not have an `ssl` parameter, as indicated in PostgreSQL [doc](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING:~:text=%2Dencrypted%20connection.-,sslmode,-This%20option%20determines). 

In dbt-redshift 1.5, we switched to using `redshift_connector`, which accepts `verify-ca`, and `verify-full` as valid `sslmode` inputs, and has a `ssl` parameter of `True` or `False`, according to redshift [doc](https://docs.aws.amazon.com/redshift/latest/mgmt/python-configuration-options.html#:~:text=parameter%20is%20optional.-,sslmode,-Default%20value%20%E2%80%93%20verify). 

For backward compatibility, dbt-redshift now supports valid inputs for `sslmode` in `psycopg2`. We've added conversion logic mapping each of `psycopg2`'s accepted `sslmode` values to the corresponding `ssl` and `sslmode` parameters in `redshift_connector`.

The table below details accepted `sslmode` parameters and how the connection will be made according to each option:

`sslmode` parameter | Expected behavior in dbt-redshift | Actions behind the scenes
-- | -- | --
disable | Connection will be made without using ssl | Set `ssl` = False
allow | Connection will be made using verify-ca | Set `ssl` = True &  `sslmode` = verify-ca
prefer | Connection will be made using verify-ca | Set `ssl` = True &  `sslmode` = verify-ca
require | Connection will be made using verify-ca | Set `ssl` = True &  `sslmode` = verify-ca
verify-ca | Connection will be made using verify-ca | Set `ssl` = True &  `sslmode` = verify-ca
verify-full | Connection will be made using verify-full | Set `ssl` = True &  `sslmode` = verify-full

When a connection is made using `verify-ca`, will look for the CA certificate in `~/redshift-ca-bundle.crt`.

For more details on sslmode changes, our design choices, and reasoning &mdash; please refer to the [PR pertaining to this change](https://github.com/dbt-labs/dbt-redshift/pull/439).

### `autocommit` parameter

The[ autocommit mode](https://www.psycopg.org/docs/connection.html#connection.autocommit) is useful to execute commands that run outside a transaction. Connection objects used in Python must have `autocommit = True` to run operations such as `CREATE DATABASE`, and `VACUUM`. `autocommit` is off by default in `redshift_connector`, but we've changed this default to `True` to ensure certain macros run successfully in your dbt project.

If desired, you can define a separate target with `autocommit=True` as such:

<File name='~/.dbt/profiles.yml'>

```yaml
profile-to-my-RS-target:
  target: dev
  outputs:
    dev:
      type: redshift
      ...
      autocommit: False
      
  
  profile-to-my-RS-target-with-autocommit-enabled:
  target: dev
  outputs:
    dev:
      type: redshift
      ...
      autocommit: True
  ```
</File>

To run certain macros with autocommit, load the profile with autocommit using the `--profile` flag. For more context, please refer to this [PR](https://github.com/dbt-labs/dbt-redshift/pull/475/files).


### Deprecated `profile` parameters in 1.5

- `iam_duration_seconds`

- `keepalives_idle`

### `sort` and `dist` keys

Where possible, dbt enables the use of `sort` and `dist` keys. See the section on [Redshift specific configurations](/reference/resource-configs/redshift-configs).



<VersionBlock firstVersion="1.2">

#### retries

If `dbt-redshift` encounters an operational error or timeout when opening a new connection, it will retry up to the number of times configured by `retries`. If set to 2+ retries, dbt will wait 1 second before retrying. The default value is 1 retry. If set to 0, dbt will not retry at all.

</VersionBlock>
