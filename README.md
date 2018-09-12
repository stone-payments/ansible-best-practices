# Ansible Best Practices

Ansible Best Practices by Stone Payments team

## Overview

The main reference for ansible best practices is the official documentation: [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html). Everyone working with ansible **must** read it.

This guide replicates some (not all) of the content on the official guide, the points we consider most important, to emphasize them. We also add here many other practices and tips that we consider important and useful.

## Contributing

PRs are most welcome!

We strongly suggest that each PR addresses or adds **only one point/practice**, so we can discuss and approve/change/deny each practice individually.

Keep in mind that while this is a place where we can be specific about Stone CO's context and practices, we should try to make this guide as useful and understandable for public release as possible. THIS IS A PUBLIC REPO.

## Best Practices

### Always Name Tasks

*From the official docs.*

"*It is possible to leave off the ‘name’ for a given task, though it is recommended to provide a description about why something is being done instead. This name is shown when the playbook is run.*"

Do it. Even for debug, include and import, set_fact, and others that you might consider "minor" tasks. When the playbook gets large, and you are trying to debug or find a single task among many others, names are a must have.

Also it may be useful to add variables to the task names, specially in tasks that are included with an outer loop (loop or with_items is not in the task, but in a outer level where it is included). Don't overdue it, limit that to cases where there is no other way to disambiguate the tasks on the playbook run. See [this example](examples/always-name-tasks/playbook.yml) inside this repo.

### Directory Layout

*From the [official docs](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout).*

We actually recomend more the [alternate directory layout](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#alternative-directory-layout), to have environments split as different inventories. 

Note that you can have a *group_vars* both on the playbook level **and** inside the inventory. That way you can specify group_vars that apply to all envinronments in the root group_vars directory. But note that any variable set in playbook level group_vars overrides the ones in the inventory. See the docs on [Variable Precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#id19). Here's a version of the alternate directory layout showing this case:

```txt
group_vars/         # Will apply to the playbooks in root dir (like site.yml below)
    group1          #  You can specify vars that apply to all environments. Note though
    group2          #  they override group_vars inside inventories if in both places
inventories/
   production/
      hosts               # inventory file for production servers
      group_vars/
         group1           # here we assign variables to particular groups
         group2           # ""
      host_vars/
         hostname1        # if systems need specific variables, put them here
         hostname2        # ""

   staging/
      hosts               # inventory file for staging environment
      group_vars/
         group1           # here we assign variables to particular groups
         group2           # ""
      host_vars/
         stagehost1       # if systems need specific variables, put them here
         stagehost2       # ""

library/
module_utils/
filter_plugins/

site.yml
webservers.yml
dbservers.yml

roles/
    requirements.yml     # ansible-galaxy requirements file, keep inside the roles dir
    common/
    webtier/
    monitoring/
    fooapp/
```

### Environment differentiation (dev/stg/prod) and consistency

*Official docs [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#how-to-differentiate-staging-vs-production) and [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#staging-vs-production).*

With the above alternate directory layout allows you to have the **same playbook for all your environments**, with differences between them managed by using different inventories and associated group_vars/host_vars sets for them. That's a **MUST HAVE** to keep our environments consistent.

*But then how do I develop/test code in dev and stagin without affecting production?* Well, obviously, use branches for that, and merge them in a dev -> staging -> prod flow.

### Use a consistent ansible repo naming scheme (and tag the repos!)

For a big environment like ours with many different projects, roles, and teams working on ansible code, that is key to foster collaboration and keep things sanely organized. We are also tagging

Use any scheme you want but keep it consistent. In our case we are adopting:

- Roles: **ansible-rolename**
  - tags: ansible, ansible-role
- Infrastructure playbooks: **infra-productname**
  - tags: ansible, ansible-project

Note that for the main (playbook) repos we are aggregating by product. The product may be composed by many applications but we code all its infra in the same repo. Also note that we avoid the name ansible on that repo 'cause eventually the product's infra may be managed by other tools (terraform, puppet), or many of them combined.

### Use ansible-galaxy for role dependencies management

See [Installing multiple roles from a file](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file).

Don't use gilt, use ansible-galaxy. For those wanting to download the roles with the .git directory intact (so you can develop/fix the role inside the downloaded roles/ dir) the coming 2.6 version of ansible adds a -g option to ansible-galaxy to keep the .git dir.

```bash
$ ansible-galaxy install --help
Usage: ansible-galaxy install [options] [-r FILE | role_name(s)[,version] | scm+role_repo_url[,version] | tar_file(s)]

Options:
...
  -g, --keep-scm-meta   Use tar instead of the scm archive option when
                        packaging the role
...
```

Also, keep the requirements.yml **inside the roles subdir**. That is necessary for Ansible Tower/AWX correctly downloading the roles. 

## License

MIT.
