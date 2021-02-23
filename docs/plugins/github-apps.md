# Using GitHub Apps for Backend Authentication

Backstage can be configured to use GitHub Apps for backend authentication. This
comes with advantages such as higher rate limits and that Backstage can act as
an application instead of a user or bot account.

It also provides a much clearer and better authorization model as a opposed to
the OAuth apps and their respective scopes.

## Caveats

- It's not possible to have multiple Backstage GitHub Apps installed in the same
  GitHub organization, to be handled by Backstage. We currently don't check
  through all the registered GitHub Apps to see which ones are installed for a
  particular repository. We only respect global Organization installs right now.
- App permissions is not managed by Backstage. They're created with some simple
  default permissions which you are free to change as you need, but you will
  need to update them in the GitHub web console, not in Backstage right now. The
  permissions that are defaulted are `metadata:read` and `contents:read`.
- The created GitHub App is private by default, this is most likely what you
  want for github.com but it's recommended to make your application public for
  GitHub Enterprise in order to share application across your GHE organizations.

A GitHub app created with `backstage-cli create-github-app` will have read
access by default. You have to manually update the GitHub App settings in GitHub
to grant the app more permissions if needed.

### Using the CLI (public GitHub only)

You can use the `backstage-cli` to create a GitHub App using a manifest file
that we provide. This gives us a way to automate some of the work required to
create a GitHub app.

You can read more about the
[`backstage-cli create-github-app` method](../cli/commands.md#create-github-app).

Once you've gone through the CLI command, it should produce a YAML file in the
root of the project which you can then use as an `include` in your
`app-config.yaml`. You can go ahead and
[skip ahead](#including-in-integrations-config) if you've already got an app.

### GitHub Enterprise

You have to create the GitHub Application manually using these
[instructions](https://docs.github.com/en/free-pro-team@latest/developers/apps/creating-a-github-app)
as GitHub Enterprise does not support creation of apps from manifests.

Once the application is created you have to generate a private key for the
application and place it in a YAML file.

The YAML file must include the following information. Please note that the
indentation for the `privateKey` is required.

```yaml
appId: 1
clientId: client id
clientSecret: client secret
webhookSecret: webhook secret
privateKey: |
  -----BEGIN RSA PRIVATE KEY-----
  ...Key content...
  -----END RSA PRIVATE KEY-----
```

### Including in Integrations Config

Once the credentials are stored in a YAML file generated by `create-github-app`,
or manually by following the [GitHub Enterprise](#gitHub-enterprise)
instructions, they can be included in the `app-config.yaml` under the
`integrations` section.

Please note that the credentials file is highly sensitive and should NOT be
checked into any kind of version control. Instead use your preferred secure
method of distributing secrets.

```yaml
integrations:
  github:
    - host: github.com
      apps:
        - $include: example-backstage-app-credentials.yaml
```