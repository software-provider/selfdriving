---
title: "How infrastructure as code improves DevOps"
date: 2022-12-08
redirect_to: https://tailscale.com/learn/infrastructure-as-code/
---

<xeblog-conv name="Cadey" mood="coffee">This article wasn't completly
written by me. This article was written by an external contractor at
Tailscale and I was asked to suggest technical fixes for it. I ended
up rewriting most of the article. This is the result. Many thanks to
Hrittik Roy for the base draft that this was built upon.</xeblog-conv>

If you're a DevOps engineer looking to advance your career, you've
probably heard of infrastructure as code (IaC). Infrastructure as code
has emerged as a key component of the DevOps movement, which aims to
improve communication and collaboration between development and
operations teams.

Among the benefits of IaC are the ability to manage infrastructure
declaratively and version control infrastructure changes, as well as
collaborate on those changes with other team members.

In this article, you'll learn about the fundamentals of IaC and how it
can help you build an infrastructure as code pipeline. You'll also
look at how you can use some popular IaC tools to help you implement
automatic and versioned infrastructure provisioning.

### How does infrastructure as code improve DevOps?

Infrastructure provisioning has always been a challenge, dating back
to the days when the only way to increase capacity was to either add
more servers to your data center rack or upgrade the servers in that
rack. Provisioning new servers was a time-consuming and error-prone
process that used to take weeks or even months, sometimes with frantic
trips to places like eBay to help make ends meet.

When cloud computing entered the picture, it also came with an API
that allowed you to provision thousands of instances in seconds and
eliminate the physical dependency of managing the logistics of your
hardware. Teams began communicating with the API, either by portal or
by code, to create instances as needed without incurring any CapEx
(capital expenditures, aka purchasing physical goods).

Unfortunately, this came with some significant drawbacks.
Infrastructure knowledge remained in the hands of a single person or a
small group of people, and transferring this knowledge was difficult
when new members joined or a key employee left the organization.
Things being siloed into a few people or even one person invited the
"bus factor" into the equation. If any of those important people left
the company, that could spell disaster.

Moreover, even with the advantages that cloud computing offered,
infrastructure provisioning remained an error-prone and slow, manual
process. For example, human error issues crept in when changes to the
running environment were not recorded or communicated to the
appropriate people. Documentation of the infrastructure setup became
out of date the moment it was written and was regarded with the same
casual loyalty as scribbling on a napkin to kill time. It was a mess.

Compliance and audit issues also tended to arise as a wide range of
services were running and being updated without a blueprint of the
infrastructure; you can't know if you are patched against a security
issue unless you know what machines are potentially affected.

The advent of IaC meant infrastructure provisioning became much
easier, faster, and more reliable, and many of the shortcomings
mentioned above were eliminated. You could even say that IaC brought
the same benefits to infrastructure that DevOps brought to software
development.

Specifically, you can continuously create, destroy, and update your
infrastructure using the same process and tools you use for your
application code as dictated by your DevOps practices. This means that
using IaC gives your infrastructure the same benefits that a DevOps
workflow gives your code. You can also use the same collaboration and
review tools to track your infrastructure changes as you do your code
changes. This makes it much easier to manage and understand your
infrastructure, and to identify and fix problems quickly.

IaC also makes it easier to automate infrastructure provisioning,
saving a lot of time and effort. It can even eliminate the need for
manual intervention altogether. This can greatly improve the speed and
reliability of your infrastructure provisioning process and make it
much easier to scale your infrastructure up or down as needed. Need to
spin up more servers? Add a line to the code. Need to spin those
servers down? Remove that line from the code. It's really that easy.

### Overview of infrastructure as code

A clear understanding of what IaC tools bring to the table can help
teams and enterprises understand its significance. Among many
benefits, the ones that stand out are cost savings, agility, and less
risky deployments.

#### Cost

Saving money is a key consideration for organizations, and IaC is a
good way to go about it. When you use IaC tools, the knowledge of
infrastructure is stored in code as opposed to in people; as such, you
can run efficiently with smaller infrastructure provisioning and
management teams. Furthermore, because these tools provide consistent
and automated deployments, there are savings in the form of reduced
error and high availability.

Large businesses can lose millions of dollars for every minute of
downtime, so avoiding downtime due to misconfiguration issues is a
sure way to save money. When you use IaC tools, you also have less
overhead because the configuration code is easy to maintain, and you
can understand the dependencies between different resources by looking
at the source.

Finally, the ability to deploy code and infrastructure quickly and in
a repeatable manner allows you to reach the market faster because your
focus is on the core business logic rather than the error-prone task
of manual management and provisioning of infrastructure.

#### Agility

Modern software delivery practices are all about speed and agility.
IaC can allow teams to rapidly adapt to traffic spikes by
automatically provisioning and configuring resources as needed. For
example, say you're running a cluster with five nodes and receiving
ten thousand requests per second. Suddenly there's a spike of nineteen
thousand requests per second. If you were to scale up manually, adding
and configuring the necessary capacity would take hours. With the
appropriate IaC tools, you simply scale up the cluster to ten nodes,
and the doubled capacity would be available almost immediately.

The most helpful part is that changing a few parameters can help you
generate QA, preprod, and production deployments without the hassle of
starting from scratch.

#### Risk

Secure services are very important when dealing with applications and
data. Configuring these services is often difficult to do without
introducing security vulnerabilities.

Infrastructure as code can help mitigate these risks by automating the
provisioning of these services, which can help ensure that they're
configured securely and consistently. You don't need to open an
interface to your portal to configure things manually and worry about
hackers sneaking in through back doors. Also, because these processes
are now automated, they can be easily repeated should something go
wrong.

Finally, IaC can help make setting up new servers easier and more
repeatable. This can help ensure that new servers are configured
correctly and securely from the outset.

### IaC methods and protocols

No one tool can be used to perform everything, so having specific
tools that are suitable for the task at hand is always going to be
more efficient. That being said, infrastructure as code can be broadly
classified into a few categories: imperative or declarative, and push
or pull.

#### Imperative vs. declarative

Imperative IaC is where you write code that explicitly details every
step required to provision your infrastructure. This is often referred
to as *procedural code*. An example would be using a shell script to
provision and configure a virtual machine.

By contrast, declarative IaC is where you write code that specifies
the desired state of your infrastructure. Using a tool like Terraform
to configure and deploy a set of virtual machines or container
orchestrators like Kubernetes is an example of this. You specify which
provider you want the cluster in and supply the tool with plug-ins and
authentication tokens. After you apply the changes, the tool takes
care of the rest.

The main difference between the two approaches is that with imperative
IaC, you have to write code to specifically detail every step required
to provision your infrastructure. This can be time-consuming and
error-prone.

With declarative IaC, you only have to write code to specify the
desired state of your infrastructure, and the IaC tool figures out
what it needs to do in order to meet that state. As this is less
time-consuming and error-prone, most popular tools (such as Terraform
and Pulumi) use a declarative approach.

#### Push vs. pull configuration

The push configuration works by sending configuration files to servers
from a central repository. The approach is very simple and
straightforward to implement. All you need is a central repository and
a method to move files from the repository to the servers.

The main disadvantage of the push approach is that it can be slow and
inconvenient if you have a large number of servers, as any changes to
the configuration must be pushed to all of the servers.

The pull configuration means that the servers check the central
repository for changes on a regular basis and download the most recent
configuration files. When you have a large number of servers, the main
advantage of this approach is that it's much faster and easier to
manage. This is due to the fact that you need to make changes in only
one place (the central repository), and the servers will automatically
pull them down.

The main disadvantage of this approach is that it's more difficult to
set up (most IaC tools are designed with the push model in mind
because it's easier to sell to people that way) and may cause conflict
if changes are made in the server before the pull is completed.
However, this is unlikely to be an issue because playing in a live
environment is not advised and tends to be restricted.

The pull method is more popular and efficient because it allows for
testing prior to infrastructure deployment. It also allows for
centralized storage of your configuration, reducing conflict and
serving as a single source of truth that your team can use as a
reference.

### Common tools for infrastructure as code

As IaC has grown in popularity, many projects have emerged from
enterprises and individual contributors serving different use cases.
You don't need to research all of them to get started; below, you'll
find some of the top IaC tools available on the market right now.

All of the tools listed below have an open source version as well as
an enterprise edition that you can use for your business needs; the
paid versions come with additional features, support, and security.

#### Terraform

Hashicorp created the most popular project on the list. Unlike the
others, [Terraform](https://www.terraform.io/) (which is written in
Go) is a tool for provisioning, managing, and configuring
infrastructure resources as opposed to configuration management.
System administrators and DevOps professionals frequently use it.
Terraform is open source and therefore completely free to use. It's
simple to learn and has widespread community support.

The tool supports a declarative infrastructure configuration written
in HashiCorp Configuration Language (HCL) and can produce immutable
infrastructure without the use of an agent. Terraform allows
developers to easily provision entire cloud landscapes, including
VPCs, compute instances, and DNS entries. It also works with a number
of well-known cloud providers like AWS, Azure, and GCP.

**Advantages**:
  - It enables users to define and provision a data center
    infrastructure using a high-level configuration language known as
    HCL.
  - It's capable of managing both popular service providers and custom
    in-house solutions.
  - It's open source, providing a community in which users can
    collaborate and contribute modules for popular services.

**Disadvantages**:
  - The learning curve can be steep for those who are not familiar
    with IaC or HCL.
  - When provisioning large infrastructures, the tool can be slow.
  - The state files can be difficult to manage when collaborating with
    a team, but there are options to store these state files in shared
    spaces, such as AWS S3.
  - Writing and using modules requires you to run `terraform init`
    many times in order to modularize your configuration. There are no
    functions or equivalent concepts in Terraform. Only modules. This
    can scale badly.

#### Pulumi

[Pulumi](https://www.pulumi.com/) is an IaC tool written in Go. It
does everything that Terraform can do, but allows users to write their
provisioning instructions in languages like Go, TypeScript, or Python
instead of having to use bespoke configuration languages like YAML or
HCL.

Writing your configuration in a normal programming language allows
users to let their IDE have _deep integration_ into the attributes and
parameters of a given resource, unlike Terraform, where users are
advised to have the documentation open constantly while developing
their infrastructure manifests. It works with major infrastructure
providers such as AWS, Microsoft Azure, Google Cloud, and any
Kubernetes cluster.

**Advantages**:
  - Writing your own code in a real programming language makes it
    easier to context switch toward maintaining infrastructure if you
    are in a programming role.
  - Modularization is far easier than with Terraform. Programming
    languages typically have native facilities for modularization
    that can work a lot better than with Terraform's modules concept.
  - Pulumi is also open source code and widely loved by the developer
    community.

**Disadvantages**:
  - Using a fully Turing-complete programming language means that
    there is the capability of creating an infrastructure plan that
    will not terminate.
  - Bugs in your infrastructure code can result in large unexpected
    amounts of infrastructure spend.
  - Pulumi isn't as widely adopted as Terraform, so some
    infrastructure providers may not be available.

### Configuration management tools

Provisioning your infrastructure isn't the end of the story. Once you
have the servers created, you will need to configure them to do what
you want. This is also a process traditionally fraught with manual
human intervention and wishful thinking as documentation. These tools
are the best in class for letting you declare what you want your
computers to contain, then setting them off to make sure it happens.

#### Ansible

Ansible is another well-known configuration management and
orchestration tool. Created by Michael DeHaan in 2012, it was
purchased by Red Hat in 2015. Ansible playbooks are written in the
YAML language. It communicates between nodes using SSH and doesn't
require agents to be installed on remote nodes. It's written in
Python, and Ansible modules and plug-ins can be easily extended by
developers.

It's worth noting that Ansible was primarily created as a
configuration management tool. Support for having Ansible manage
infrastructure such as AWS is a fairly recent innovation. It's kind of
both an IaC and a configuration management tool, but it's more mature
as a configuration management tool.

For configuration management, the tool employs push methodology in
which configurations are pushed from a central server (or developer
laptop) to all nodes or target hosts. Community support is strong for
Ansible, with more than 35,000 modules and a lot of forums.

**Advantages**:
  - It's agentless, so there's no need to install any additional
    software on your servers.
  - It's very powerful and can be used to manage complex deployments.
  - It's open source and therefore cost effective.

**Disadvantages**:
  - Because Ansible is agentless, it relies on SSH for communication
    with remote servers, which can be a security concern. You can use
    a solution like [Tailscale](https://tailscale.com/) to protect
    your connection and harden your connections.
  - Ansible is not as widely used as some of the other options, so
    less community support is available.
  - It's challenging to learn and use for those without a good
    understanding of system administration and DevOps concepts.

#### Chef

[Chef](https://www.chef.io/) is a configuration management tool for
installing and managing software on existing servers. Its
configuration language is a custom domain-specific language (DSL)
based on Ruby. The tool uses a pull-based approach to sync changes and
produces mutable changes. Chef is popular for its integration with
cloud providers.

The tool comes with issue detection in preproduction environments and
provides complete security with compliance visibility across all
stages.

**Advantages**:
  - Organizations can avoid costly security issues by testing for
    security and compliance early on in the development process.
  - Automating configuration defined in code can help embed security
    tests in the delivery process.
  - Organizations can automate across heterogeneous infrastructure to
    ensure servers return to their desired state.

**Disadvantages**:
  - It doesn't support push-based IaC.
  - It's a complex tool that can be difficult to learn if you're not
    already familiar with Ruby and procedural coding.

#### Puppet

[Puppet](https://puppet.com/) is a free and open source configuration
management system that can aid in the automation of repetitive tasks,
the deployment of applications, and the management of system
configurations across a group of servers.

This tool created by Puppet Labs is written in Ruby. The Apache
License 2.0 governs its distribution. Puppet uses the declarative
approach and is appropriate for Unix-like and Microsoft Windows
systems.

**Advantages**:
  - Puppet is declarative, meaning that you only need to describe the
    desired state of your infrastructure.
  - It's idempotent, meaning that it will only make changes to your
    infrastructure in order to reach the desired state. This means
    that you can run Puppet as often as you like without fear of
    breaking things.
  - The tool is powerful and flexible, allowing you to manage
    everything from simple files and packages to complex networks and
    cloud deployments.

**Disadvantages**:
  - Puppet can be complex to learn and use, especially if you're not
    familiar with programming concepts.
  - Sometimes it can be slow, especially when you're managing a large
    infrastructure.
  - Procedures can be difficult to debug.

#### SaltStack

[SaltStack](https://github.com/saltstack/salt) is a powerful open
source configuration management and remote execution tool. It can be
used for many processes, such as managing server deployments and
automating tasks. It follows a declarative approach, meaning that you
describe the desired state of your system and SaltStack takes care of
the rest.

**Advantages**:
  - It's a platform with a simple solution that works very effectively.
  - It is powerful and flexible, as it automatically detects issues
    and forces the system to return to the desired state.
  - It ensures critical infrastructure is always available.

**Disadvantages**:
  - Community support for the tool is limited.
  - The learning curve is steep when you're getting started.
  - It's not well suited for large environments as there is limited
    support for hardware.

#### CFEngine

[CFEngine](https://cfengine.com/) is another powerful configuration
management tool for automating system administration tasks. It allows
you to define and manage your system's configuration as well as
automate complex tasks such as deployments and system updates.

The tool is used by companies such as Samsung and DHL.

**Advantages**:
  - CFEngine supports patch management on your infrastructure.
  - Infrastructure hardening is also supported.
  - CFEngine helps with compliance automation by performing audits and
    checking systems every five minutes to ensure they're always
    enforced and adhering to your compliance framework.

**Disadvantages**:
  - There's no native integration for cloud as with the other tools.
  - There's no existing push mechanism.

#### Nix and NixOS

[NixOS](https://nixos.org) is a next-generation set of tools that lets
you describe _precisely_ what a server should run and all of the
settings that are normally specified by all the bespoke configuration
languages in use on an average production-worthy Linux system. By
having deep integration with its package manager Nix, it can also
automatically rebuild your custom software as a part of the deploy
process. This allows you to describe _exactly_ what you want and have
the confidence to know _exactly_ what is going on.

Packages and system configurations are defined in the same language
(also called Nix), meaning that your custom packages can be as deeply
integrated into the system as with the services that NixOS exposes
with its options system. Your configuration for your custom services
can be expressed right next to your nginx configuration to expose them
to the internet.

**Advantages**:
  - NixOS gives you the ability to know _everything_ that is going on
    with a system. This makes triaging security issues trivial. Once
    the vulnerable package is identified, it’s trivial to create a
    mapping of every other package that depends on it. This helps you
    evaluate the "splash damage" of remediation efforts.
  - Nix is a _functionally pure_ package manager in the sense that
    package builds cannot randomly access the internet or random parts
    of the system. This means that even if one of your dependencies
    has a compromised build script, it cannot phone home with secret
    credentials.
  - NixOS is one of the largest open source repositories on GitHub and
    has been steadily gaining adoption as years go by.
  - The architecture of NixOS makes it difficult for most attack tools
    to work because it doesn't place critical system files in standard
    locations. This is security by obscurity at best, but it does
    work.

**Disadvantages**:
  - Nix the package manager, Nix the configuration language, and NixOS
    the operating system are all different things but have nearly
    identical names. This can lead to ontological confusion.
  - The Nix language syntax looks a bit like Haskell and JSON thrown
    into a blender, with semicolons at the ends of some statements
    like C. This can prove difficult for newcomers at first, but
    usually is not a practical issue in the long run.
  - Nix and NixOS don't define push tooling in its core, but there are
    facilities for having a machin[<35;84;32Me periodically pull its
    configuration and rebuild itself from a shared Git repository.

## Frequently Asked Questions

Here are some common questions about infrastructure as code and their
answers.

**How does IaC improve DevOps?**
      
Using IaC makes it easier to automate infrastructure provisioning,
which saves time and effort, makes scaling much simpler, and reduces
human error. It can give you the ability to version control
infrastructure and manage it declaratively, further increasing
efficiency. It also simplifies sharing information about your
infrastructure, since that knowledge is stored in code, as opposed to
in people.

**What’s the difference between imperative IaC and declarative IaC?**
      
With imperative IaC, you are required to write code (often called
*procedural code*) to specify each step that is required to provision
your infrastructure, which can be a time-consuming and error-prone
process. By contrast, with declarative IaC, you write code only to
specify the desired state of your infrastructure, and the IaC tool
determines what it needs to do in order to meet that state.

**What is a push configuration vs. a pull configuration?**

The push configuration works by sending configuration files to servers
from a central repository. This approach is simple to implement, but
it can be slow and inconvenient if it involves pushing configuration
changes to a lot of servers.

In a pull configuration, servers check the central repository for
changes on a regular basis and download the most recent configuration
files. This is faster and easier to manage than the push
configuration, but it can be more difficult to set up, and conflicts
may arise if changes are made in a server before one of its pulls is
completed.

### Final thoughts

This article has covered the fundamentals of infrastructure as code:
what it is, why it matters, and how to get started. With knowledge of
some of the more popular tools on the market, you should now be better
equipped to begin managing your infrastructure in a more efficient and
effective manner. Remember that IaC is just one of many tools in your
DevOps toolbox and that protecting the tools from external threats is
also critical.

Tailscale is a cloud-based solution that integrates with your SSO to
secure access to your servers, cloud instances, and tools. It uses
WireGuard® to encrypt your connection between you and your servers.
This can augment ex[<35;83;32Misting infrastructure as well as make
it easy to extend into more complicated infrastructure setups.
Tailscale also provides a [Terraform
provider](https://tailscale.com/blog/terraform/) to allow you to
automate things further. Want to automatically create authentication
keys for new servers you provision? You can do that with Tailscale's
Terraform provider.

Create an account, authenticate, and [download Tailscale to try our
zero config VPN](https://tailscale.com/download/).
