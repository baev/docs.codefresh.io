---
title: "Git tokens for GitOps"
description: "Understand Git tokens and scopes required for GitOps"
group: security
redirect_from:
  - /docs/administration/git-tokens/ 
  - /docs/reference/git-tokens/ 
toc: true
---



Codefresh requires two types of Git tokens for authentication in GitOps, a Git Runtime token, and a Git user token. The Runtime and user tokens are both Git access tokens, that Codefresh uses for different purposes. See [Git Runtime tokens versus Git user tokens in Codefresh](#git-runtime-tokens-versus-git-user-tokens-in-codefresh). 
* The [Git Runtime token](#git-runtime-token-scopes) is mandatory for every GitOps Runtime. It must be provided during the Runtime installation, and can be a service/Robot account token.
* The [Git user token](#git-user-access-token-scopes) is an access token that is unique to every user in the Codefresh platform. It is required after installation for every Runtime which the user has access to. 


Users can also create and use Git tokens with custom scopes for both GitOps Runtimes and for Git repositories associated with the Runtimes that they need to access. See [Git user tokens with custom scopes](#git-user-tokens-with-custom-scopes).

## Git Runtime tokens versus Git user tokens in Codefresh
The table below summarizes the main differences between the Git Runtime and user tokens in Codefresh.

{: .table .table-bordered .table-hover}
|                            | Git Runtime token                  | Git user token         |
| -------------------------- | ---------------------          | ------------------ |
| Usage                      | {::nomarkdown}<ul><li>_During installation_, to create the Git repository and install the GitOps Runtime.</li><li>_After installation_, used by:<ul><li>Argo CD to clone the Git repos, pull changes, and sync to the K8s cluster.</li><li> Argo Events to create web hooks in Git repositories.</li><li><code class="highlighter-rouge">cap-app-proxy</code> to clone the Shared Configuration Repository</li></ul> {:/} | Authenticate and authorize user actions in Codefresh UI and CLI to Git repositories for every provisioned GitOps Runtime. <br>Users can view and manage the Git user tokens assigned to the Runtimes in the [Git Personal Access Token](https://g.codefresh.io/2.0/user-settings){:target="\_blank"} page.  |
| Created                    | Before Runtime installation; see [required scopes for Git Runtime tokens](#git-runtime-token-scopes).   | After Runtime installation; see [required scopes for Git user tokens](#git-user-access-token-scopes).
| Managed by                    | Admin at account-level                    | User   |
| Associated Account Type    | (Recommended) [Service account or robot account](#use-a-servicerobot-account-for-gitops-runtimes) | User account    |


## Git Runtime token scopes
The table below lists the scopes required for Git Runtime tokens for the different Git providers. You can also create a Git Runtime token with custom scopes and [add it directly to the `values.yaml` file](#git-runtime-token-in-valuesyaml).


| Git provider                  | Required scopes for Git Runtime token           | 
| ---------------------------- | ------------------------------ | 
| GitHub and GitHub Enterprise |{::nomarkdown}<ul><li>Classic:<ul><li><code class="highlighter-rouge">repo</code></li><li><code class="highlighter-rouge">admin:repo_hook</code></li></ul><li>Fine-grained:<ul><li>Repository access: <code class="highlighter-rouge">All repositories</code> or <code class="highlighter-rouge">Only select repositories</code></li><li>Repository permissions:<ul><li>Administration: <code class="highlighter-rouge">Read and write</code></li><li>Contents: <code class="highlighter-rouge">Read and write</code></li><li>Metadata: <code class="highlighter-rouge">Read-only</code></li><li>Webhook: <code class="highlighter-rouge">Read and write</code></li></ul></li></ul></li></ul>{:/}|
| GitLab Cloud and GitLab Server       |{::nomarkdown}<ul><li><code class="highlighter-rouge">read_api</code></li><li><code class="highlighter-rouge">read_repository</code></li></ul> {:/}         |             
| Bitbucket Cloud and Bitbucket Server | {::nomarkdown} <ul><li>Account: <code class="highlighter-rouge">Read</code></li><li>Workspace membership: <code class="highlighter-rouge">Read</code></li><li>Webhooks: <code class="highlighter-rouge">Read and write</code></li><li>Repositories: <code class="highlighter-rouge">Write, Admin </code></li></ul>{:/}|

### Git Runtime token in values.yaml

You also have the option to directly add your Git Runtime token, or a reference to a secret that contains the Git Runtime token, to `values.yaml` (typically the latter).  

To skip token validation both during installation and upgrade in this scenario, add the `skipValidation` flag to `values.yaml`. 

```yaml
installer:
  skipValidation: true
```

{{site.data.callout.callout_warning}}
**IMPORTANT**  
If you set the flag to skip validation, _the onus is on you to provide a valid and secure token_.  
{{site.data.callout.end}}



## Git user access token scopes
The table below lists the scopes required for Git user access tokens for the different Git providers. 
As with the Git Runtime token, you can create and use Git user tokens with custom scopes per GitOps Runtime, and per Git repository to which the Runtime has access. 


| Git provider                  | Required scopes for Git user token          | 
| ---------------------------- | ------------------------------ | 
| GitHub and GitHub Enterprise |{::nomarkdown}<ul><li>Classic:<ul><li><code class="highlighter-rouge">repo</code></li></ul><li>Fine-grained:<ul><li>Repository access: <code class="highlighter-rouge">All repositories</code> or <code class="highlighter-rouge">Only select repositories</code></li><li>Repository permissions:<ul><li>Contents: <code class="highlighter-rouge">Read and write</code></li><li>Metadata: <code class="highlighter-rouge">Read-only</code></li></ul></li></ul></li></ul>{:/}|
| GitLab Cloud and GitLab Server       |{::nomarkdown}<ul><li><code class="highlighter-rouge">write_repository</code> (includes <code class="highlighter-rouge">read_repository</code>) </li><li><code class="highlighter-rouge">api_read</code></li></ul> {:/}  |
| Bitbucket Cloud and Bitbucket Server | {::nomarkdown} <ul><li>Account: <code class="highlighter-rouge">Read</code></li><li>Workspace membership: <code class="highlighter-rouge">Read</code></li><li>Webhooks: <code class="highlighter-rouge">Read and write</code></li><li>Repositories: <code class="highlighter-rouge">Write, Admin </code></li></ul>{:/}|


### Git user tokens with custom scopes 
Codefresh validates Git user tokens and their associated scopes when authorizing Git actions for the Runtime.  


If you require custom scopes in Git user tokens that don't meet the default Codefresh requirements, you can create Git user tokens with custom scopes. You may want to have Git user tokens without `admin` scopes, or use the new fine-grained tokens for GitHub (currently not officially supported by Codefresh). 

Codefresh provides the `skipGitPermissionValidation` flag which you can add to your `values.yaml` file to bypass token validation for such cases. 


```yaml
app-proxy:
  config:
    skipGitPermissionValidation: "true"
```

If you set this flag, make sure that:
1. The Git user token defined for the GitOps Runtime (the token defined for `runtime-repo-creds-secret`), has read and write access to the Shared Configuration Repository.
1. The Git user tokens for the different Git repositories associated with the Runtimes have read and write permissions to those Git repositories they expect to write to and read from.  
  Read more on configuring the repositories with multiple `repo-creds` secrets in [Argo CD Repositories](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories).

{{site.data.callout.callout_warning}}
**IMPORTANT**  
If you set the flag to skip validation, _the onus is on you to provide valid and secure tokens_. Codefresh does not validate the tokens whenever Git Runtime and Git user tokens are updated. 
{{site.data.callout.end}}

### Use same Git user tokens for multiple GitOps Runtimes
If a user has access to multiple GitOps Runtimes in the same or in different accounts in Codefresh, they can use either the same Git user token to authenticate and authorize all the Runtimes to which they have access.     

>**NOTE**  
The user must configure the Git user token for each GitOps Runtime separately.  

### Manage Git user tokens
User can manage their Git user tokens for Runtimes, as described in [Managing Git PATS]({{site.baseurl}}/docs/administration/user-self-management/manage-pats/).


## Use a service/robot account for GitOps Runtimes
For GitOps Runtime installation, we recommend using an account not related to any specific user in your organization. 
Service/robot accounts are ideal for this purpose, as they provide secure authentication, restricted permissions, and centralized management. 

You need to create a service or robot account with your Git provider, generate the Git Runtime token, and use this account exclusively to install GitOps Runtimes.

## Related articles  
[Managing Git PATs]({{site.baseurl}}/docs/administration/user-self-management/manage-pats/)  
[User settings]({{site.baseurl}}/docs/administration/user-self-management/user-settings/)  
[Secrets for GitOps]({{site.baseurl}}/docs/security/secrets/)  
[Verifying authenticity of Codefresh artifacts]({{site.baseurl}}/docs/security/codefresh-signed-artifacts/)  
