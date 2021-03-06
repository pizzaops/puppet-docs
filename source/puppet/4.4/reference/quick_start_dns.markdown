---
layout: default
title: "Quick Start » DNS"
subtitle: "DNS Quick Start Guide"
canonical: "/puppet/latest/quick_start_dns.html"
---

[downloads]: http://info.puppetlabs.com/download-pe.html
[sys_req]: ./install_system_requirements.html
[agent_install]: ./install_agents.html
[install_overview]: ./install_basic.html

Welcome to the Open Source Puppet DNS Quick Start Guide. This document provides instructions for getting started managing a simple DNS nameserver file with Puppet. A nameserver ensures that the "human-readable" names you type in your browser (for example, `example.com`) can be resolved to IP addresses that computers can read.

Sysadmins typically need to manage a nameserver file for internal resources that aren't published in public nameservers. For example, let's say you have several employee-maintained servers in your infrastructure, and the DNS network assigned to those servers use Google's public nameserver located at `8.8.8.8`. However, there are several resources behind your company's firewall that your employees need to access on a regular basis. In this case, you'd build a private nameserver (say at `10.16.22.10`), and then use Puppet to ensure all the servers in your infrastructure have access to it.

 In this exercise, you will learn how to do the following steps:

* [Write a simple module that contains a class called `resolver` to manage a nameserver file called `/etc/resolv.conf`](#write-the-resolver-class).
* [Enforce the desired state of that class from the command line of your puppet agent](#enforce-the-desired-state-of-the-resolver-class).

> Before starting this walk-through, complete the previous exercise in the [essential configuration tasks](./quick_start_essential_config.html), which is setting up [NTP](./quick_start_ntp.html). Log in as root or administrator on your nodes.

>**Note**: You can add the DNS nameserver class to as many agents as needed. For ease of explanation, this guide will describe only one agent.

### Write the `resolver` Class

Some modules can be large, complex, and require a significant amount of trial and error, while others often work right out of the box. This module will be a very simple module to write, as it contains just one class and one template.

> #### A Quick Note about Modules
>
>By default, Puppet keeps modules in an environment's [`modulepath`](./dirs_modulepath.html), which for the production environment defaults to `/etc/puppetlabs/code/environments/production/modules`. This includes modules that Puppet installs, those that you download from the Forge, and those you write yourself.
>
>**Note:** Puppet also creates another module directory: `/opt/puppetlabs/puppet/modules`. Don't modify or add anything in this directory, including modules of your own.
>
>There are plenty of resources about modules and the creation of modules that you can reference. Check out [Module Fundamentals](./modules_fundamentals.html), the [Beginner's Guide to Modules](/guides/module_guides/bgtm.html), and the [Puppet Forge](https://forge.puppetlabs.com/).

Modules are directory trees. For this task, you'll create the following files:

 - `resolver` (the module name)
   - `templates/`
      - `resolv.conf.erb` (contains template for `/etc/resolv.conf`, the contents of which will be populated after you add the class and run Puppet.)

**To write the `resolver` class**:

1. From the command line on the Puppet master, navigate to the modules directory: `cd /etc/puppetlabs/code/environments/production/modules`.

2. Run `mkdir -p resolver/templates` to create the new module directory and its templates directory.

3. Use your text editor to create the `resolver/templates/resolv.conf.erb` file.

4. Edit the `resolv.conf.erb` file to add the following Ruby code. This Ruby code is a template for populating `/etc/resolv.conf` correctly, no matter what changes are manually made to `/etc/resolv.conf`, as we will see in a later example.

        # Resolv.conf generated by Puppet

        <% [@nameservers].flatten.each do |ns| -%>
        nameserver <%= ns %>
        <% end -%>

        # Other values can be added or hard-coded into the template as needed.

5. Save and exit the file.

> That's it! You've created a Ruby template to populate `/etc/resolv.conf`.

## Add the resolv.conf File to Your Main Manifest

1. On the Puppet master, open `/etc/resolv.conf` with your text editor, and copy the IP address of your master's nameserver (in this example, the nameserver is `10.0.2.3`).

2. On the Puppet master, navigate to the main manifest: `cd /etc/puppetlabs/code/environments/production/manifests`.
3. Use your text editor to open the `site.pp` file and add the following Puppet code to the `default` node, editing your nameserver value to match the one you found in `/etc/resolv.conf`:

        $nameservers = ['10.0.2.3']

        file { '/etc/resolv.conf':
          ensure  => file,
          owner   => 'root',
          group   => 'root',
          mode    => '0644',
          content => template('resolver/resolv.conf.erb'),
        }

4. From the command line on your Puppet agent, run `puppet agent -t`.
5. From the command line on your Puppet agent, run `cat /etc/resolv.conf`. The result should reflect the nameserver you added to your main manifest in step 3.

> That's it! You've written a module that contains a class that will ensure your agents resolve to your internal nameserver.

> Note the following about your new class:
>
> * It ensures the creation of the file `/etc/resolv.conf`.
> * The content of `/etc/resolv.conf` is modified and managed by the template, `resolv.conf.erb`.

### Enforce the Desired State of the `resolver` Class

Finally, let's take a look at how Puppet will ensure the desired state of the `resolver` class on your agents. In the previous task, you set the nameserver IP address. Now imagine a scenario where a member of your team changes the contents of `/etc/resolv.conf` to use a different nameserver and can no longer access any internal resources.

1. On any agent to which you applied the `resolv.conf` class, edit `/etc/resolv.conf` to be any  nameserver IP address other than the one you desire to use.
2. Save and exit the file.
3. From the command line on your Puppet agent, run `puppet agent -t --onetime`.
4. From the command line on your Puppet agent, run `cat /etc/resolv.conf`, and notice that Puppet has enforced the desired state you specified on your Puppet master.

> That's it --- Puppet has enforced the desired state of your agent!

### Other Resources

For more information about working with Puppet and DNS, check out our [Dealing with Name Resolution Issues](http://puppetlabs.com/blog/resolving-dns-issues) blog post.

Puppet offers many opportunities for learning and training, from formal certification courses to guided online lessons. We've noted a few below. Head over to the [Learning Puppet page](https://puppetlabs.com/learn) to discover more.

* The Puppet workshop contains a series of self-paced, online lessons that cover a variety of topics on Puppet basics. You can sign up at the [learning page](https://puppetlabs.com/learn).
* Learn about [Puppet DNS](https://puppetlabs.com/learn/puppet-dns) through this online training workshop.

----------

Next: [Sudo Quick Start Guide](./quick_start_sudo.html)
