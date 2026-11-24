---
icon: lucide/rocket
---

# Add custom Specs Repository
There are three repositories in place that allow to manage the clients via git.

1. [The Ansible base collection](https://github.com/projectpotos/ansible-collection-potos.base.git) contains everything required for initial enrollment and the configuration of management triggers. The collection is publicly available on GitHub and can be used directly from the upstream repository, without requiring any modifications.
  
2. [The Ansible Potos roles collection](https://github.com/projectpotos/ansible-collection-potos.roles.git) provides a set of Ansible roles that simplify the management of Potos Linux clients. The collection is publicly available on GitHub and can be used directly from its upstream repository.

3. [The Specs repository](https://github.com/projectpotos/ansible-specs-fedora-test) contains the actual configuration and specifications for a particular Potos Linux client deployment. It is the configuration layer that connects the base collection management with the roles provided by the Potos roles collection. The specs repo can be private or public. Create and host your own copy of the example specs repository.
  
## Extend Configuration
In the `input/config.yml` add the uri and branch for the base collection.

```yaml
collection_source: git+https://github.com/projectpotos/ansible-collection-potos.base.git,main
```

Modify the specs repo information in `input/config.yml` adding the uri and branch for your copy of the specs repository.

```yaml
specs:
  url: https://git.your-organisation.com/something/specs-repo.git
  branch: main
```

In case that your specs repo is not publicly available, additional configuration for a access token is needed.
First create an access token with Scope `read_repository` and Role `Reporter`.
Then modify the `input/config.yml` as follows.

```yaml
credentials:
  specs_token:
    backend: file
    name: specs_token
    path: /etc/potos/potos_specs_token.cred

firstboot:
  on_success: logout
  firstboot:
    steps:
      - id: set_hostname
        uses: hostname/change
        input:
          ask: true
      - id: enter_specs_token
        uses: prompt/ask
        input:
          text: "Enter you specs repo token"
          fields:
            - label: Token
          validations:
            - type: required
              field: Token
              error_message: Please enter the Token.
      - id: write_specs_token
        uses: file/write
        input:
          path: /etc/potos/potos_specs_token.cred
          mode: 0600
          content: "{{ steps.enter_specs_token.value }}"
          secret: true
```

Then build a new iso using this `input/config.yml` and install it in a VM as described in [Quickstart Guide](quickstart.md).
When you first login into the freshly installed client, a dialog will ask you the enter your access token for the specs repository.
It is stored to `/etc/potos/potos_specs_token.cred` and can be modified later. The specs repository's uri and branch are stored to `/etc/potos/config.yml`.

## Test the Ansible Pull
To test if everything is setup correctly, run the following Ansible pull command.

```bash
potos-ansible-pull -r daily -v
```
