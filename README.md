# Lab - Dashy

As the lab grows in complexity with the myriad of services and applications, it is useful to have a better way to navigate than browser bookmarks and directly navigating to services - I personally forget I'm running half the things that I am.

Dashy allows setting up a dashboard with links to services and applications, and is a great way to navigate the lab.  It provides some basic authentication, service monitoring, and can be extended with custom widgets and best of all - is configured with just some YAML!

## Host Requirements

- Podman
- FirewallD

The Ansible Playbook will install all the latest versions of these packages on the target hosts.

You'll also need the dependant Ansible Collections installed:

```bash=
ansible-galaxy collection install -r ./collections/requirements.yml
```

## Using this repository

First off, the configuration in this repo is mine which means it's not likely to be usable by you - and you don't have edit access to this repo more than likely.  So, you'll need to fork this repo and then clone your forked repo to your local machine.

1. [Fork this repo](https://github.com/kenmoini/lab-dashy/fork)
2. Clone your forked repo to your local machine - `git clone https://github.com/YOUR_USERNAME/lab-pihole.git`.
3. If you'll be running the Playbook manually with the CLI then set up your inventory file - see the [examples/inventory](examples/inventory) file for an example of how to do this.  If you plan on using this with Ansible Tower you can skip creating the inventory file.
4. Next, you'll need to create the directory structure for your Site Config files.  This allows you to deploy multiple Dashy containers with multiple services to multiple hosts, and have it all atomically defined as YAML.  The directory structure for the Site Configs is as follows:

```
site_configs/
├── {inventory_hostname} (the name of the Host in your Ansible Inventory)
│   ├── {YAML FILES} (files for your Site Config, like dashboards.yml)
```

5. Create your Site Config files.  See the [examples/site_configs](examples/site_config.yml) file for examples.  You can organize the `dashy_services` variable for each inventory host however you'd like, even adding host-specific vaulted secrets or other variables in the host's site config folder.
6. Next you can run the Playbook with:

```bash=
ansible-playbook -i inventory deploy.yml
```

Since this Dashy configuration may change often, it's only appropriate to do something more GitOps-y and automate with Ansible Tower or the AAP2 Controller.

### Getting Started - Ansible Tower

Once your Site Configs are defined and any additional secrets are vaulted, you can import the Playbook into Ansible Tower and run it on a schedule and on `git push`es - handy!

Ideally as soon as you commit a change to the `main` branch of this repo, the changes will be synced to the actual services running on the hosts.  To do this, we'll use Ansible Tower to run the `deploy.yml` playbook.  Assuming you already have Ansible Tower or the AAP2 Controller installed and ready to go, you can follow these steps to set things up:

1. Create a **Credential(s)** for the systems that you will be connecting to.  You can use the `Machine` credential type for this.  You can also use the `Source Control` credential type for the Git repo that you'll be using for the configuration files.
2. Create an **Inventory** for the hosts that you'll be deploying the Dashy service containers to.  See the [examples/inventory](examples/inventory) file for an example of the Inventory structure.
3. Add the **Hosts** to the **Inventory**.
4. Create a **Project** for your fork of this Git repo.  I like to keep the names the same as the repo, so I named mine `lab-dashy`.  Make sure to check the `Update Revision on Launch` box.
5. Create a **Template** for the `deploy.yml` Playbook.  Important: Make sure to:
  - Check the `Enable Webhook` box, this will allow you to trigger the Playbook from a GitHub/GitLab webhook.
  - Check the `Privileged Escalation` box
  - Add the Credentials for the Machines, Source Control, and Vault as needed
1. Once the Template is created, you can get the `Webhook URL` and the `Webhook Key` from the **Details** page of the Template.  You'll need these to set up the Webhook in GitHub/GitLab.

Assuming you'll be using this with GitHub, you can follow these steps to set up the Webhook: https://docs.ansible.com/ansible-tower/latest/html/userguide/webhooks.html

> With this, now when you perform a `git push` to the repo to update the intended state of your managed Dashy service containers, the Playbook will be triggered and the changes will be automatically deployed to the hosts!
