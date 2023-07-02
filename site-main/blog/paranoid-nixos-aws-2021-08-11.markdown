---
title: Paranoid NixOS on AWS
date: 2021-08-11
author: Heartmender
series: nixos
tags:
 - paranix
 - aws
 - r13y
---

In [the last post](https://xeiaso.net/blog/paranoid-nixos-2021-07-18) we
covered a lot of the base groundwork involved in making a paranoid NixOS setup.
Today we're gonna throw this into prod by making a base NixOS image with it.

[Normally I don't suggest people throw these things into production directly, if
only to have some kind of barrier between you and your money generator; however
today is different. It's probably not completely unsafe to put this in
production, but I really would suggest reading and understanding this article
before doing so.](conversation://Cadey/coffee)

At a high level we are going to do the following:

- Pin production OS versions using [niv](https://github.com/nmattia/niv)
- Create a script to automatically generate a production-ready NixOS image that
  you can import into The Cloud
- Manage all this using your favorite buzzwords (Terraform,
  Infrastructure-as-Code)
- Install an nginx server reverse proxying to the [Printer facts
  service](https://printerfacts.cetacean.club/)
  
## What is an Image?

Before we yolo this all into prod, let's cover what we're actually doing.
There are a lot of conflicting buzzwords here, so I'm going to go out of my way
to attempt to simplify them down so that we use my arbitrary definitions of
buzzwords instead of what other people will imply they mean. You're reading my
blog, you get my buzzwords; it's as simple as that.

In this post we are going to create a base system that you can build your
production systems on top of. This base system will be crystallized into an
_image_ that AWS will use as the initial starting place for servers.

[So you create the system definition for your base system, then turn that into
an image and put that image into AWS?](conversation://Mara/hmm)

[Yep! The exact steps are a little more complicated but at a high level that's
what we're doing.](conversation://Cadey/enby)

## Base Setup

I'm going to be publishing my work for this post
[here](https://tulpa.dev/cadey/paranix-configs), but you can follow along in
this post to understand the individual steps here.

First, let's set up the environment with
[lorri](https://github.com/nix-community/lorri) and
[niv](https://github.com/nmattia/niv). Lorri will handle creating a cached
nix-shell environment for us to run things in and niv will handle pinning NixOS
to an exact version so you can get a more reproducible production environment.

Set up lorri:

```console
$ lorri init
Aug 11 09:41:50.966 INFO wrote file, path: ./shell.nix
Aug 11 09:41:50.966 INFO wrote file, path: ./.envrc
Aug 11 09:41:50.966 INFO done
direnv: error /home/cadey/code/cadey/paranix-configs/.envrc is blocked. Run `direnv allow` to approve its content
$ direnv allow
direnv: loading ~/code/cadey/paranix-configs/.envrc
Aug 11 09:41:54.581 INFO lorri has not completed an evaluation for this project yet, nix_file: /home/cadey/code/cadey/paranix-configs/shell.nix
direnv: export +IN_NIX_SHELL
```

[Why are you putting the `$` before every command in these examples? It looks
extraneous to me.](conversation://Mara/hacker)

[The `$` is there for two main reasons. First, it allows there to be a clear
delineation between the commands being typed and their output. Secondly it makes
it slightly harder to blindly copy this into your shell without either editing
the `$` out or selecting around it. My hope is that this will make you read the
command and carefully consider whether or not you actually want to run
it.](conversation://Cadey/enby)

Set up niv:

```console
$ niv init
Initializing
  Creating nix/sources.nix
  Creating nix/sources.json
  Importing 'niv' ...
  Adding package niv
    Writing new sources file
  Done: Adding package niv
  Importing 'nixpkgs' ...
  Adding package nixpkgs
    Writing new sources file
  Done: Adding package nixpkgs
Done: Initializing
```

[If you don't already have niv in your environment, you can hack around that by
running all the niv commands before you set up `shell.nix` like this: <br /> <pre
class="language-console"><code class="language-console">$ nix-shell -p niv --run 'niv blah'</code></pre>](conversation://Mara/hacker)

And finally pin nixpkgs to a specific version of NixOS. 

[At the time of writing this article, NixOS 21.05 is the stable release, so that
is what is used here.](conversation://Mara/hacker)

```console
$ niv update nixpkgs -b nixos-21.05
Update nixpkgs
Done: Update nixpkgs
$ 
```

This will become the foundation of our NixOS systems and production images.

You should then set up your `shell.nix` to look like this:

```nix
let
  sources = import ./nix/sources.nix;
  pkgs = import sources.nixpkgs { };
in pkgs.mkShell {
  buildInputs = with pkgs; [
    niv
    terraform
    
    bashInteractive
  ];
};
```

### Set Up Unix Accounts

[This step can be omitted if you are grafting this into an existing NixOS
configs repository, however it would be good to read through this to understand
the directory layout at play here.](conversation://Mara/hacker)

It's probably important to be able to have access to production machines. Let's
create a NixOS module that will allow you to SSH into the machine. In your
paranix-configs folder, run this command to make a `common` config directory:

```console
$ mkdir common
$ cd common
```

Now in that common directory, open `default.nix` in ~~emacs~~ your favorite text
editor and copy in this skeleton:

```nix
# common/default.nix

{ config, lib, pkgs, ... }:

{
  imports = [ ./users.nix ];
  
  nix.autoOptimiseStore = true;

  users.users.root.openssh.authorizedKeys.keys = [ "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPg9gYKVglnO2HQodSJt4z4mNrUSUiyJQ7b+J798bwD9" ];
  
  services.tailscale.enable = true;
  
  # Tell the firewall to implicitly trust packets routed over Tailscale:
  networking.firewall.trustedInterfaces = [ "tailscale0" ];
  
  security.auditd.enable = true;
  security.audit.enable = true;
  security.audit.rules = [
    "-a exit,always -F arch=b64 -S execve"
  ];
  
  security.sudo.execWheelOnly = true;
  environment.defaultPackages = lib.mkForce [];
  
  services.openssh = {
    passwordAuthentication = false;
    allowSFTP = false; # Don't set this if you need sftp
    challengeResponseAuthentication = false;
    extraConfig = ''
      AllowTcpForwarding yes
      X11Forwarding no
      AllowAgentForwarding no
      AllowStreamLocalForwarding no
      AuthenticationMethods publickey
    '';
  };
  
  # PCI compliance
  environment.systemPackages = with pkgs; [ clamav ];
}
```

[Astute readers will notice that this is less paranoid than the last post. This
was pared down after private feedback.](conversation://Mara/hacker)

This will create `common` as a folder that can be imported as a NixOS module
with some basic settings and then tells NixOS to try importing `users.nix` as a
module. This module doesn't exist yet, so it will fail when we try to import it.
Let's fix that by making `users.nix`:

```nix
# common/users.nix

{ config, lib, pkgs, ... }:

with lib;

let
  # These options will be used for user account defaults in
  # the `mkUser` function.
  xeserv.users = {
    groups = mkOption {
      type = types.listOf types.str;
      default = [ "wheel" ];
      example = ''[ "wheel" "libvirtd" "docker" ]'';
      description =
        "The Unix groups that Xeserv staff users should be assigned to";
    };
    
    shell = mkOption {
      type = types.package;
      default = pkgs.bashInteractive;
      example = "pkgs.powershell";
      description =
        "The default shell that Xeserv staff users will be given by default.";
    };
  };
  
  cfg = config.xeserv.users;

  mkUser = { keys, shell ? cfg.shell, extraGroups ? cfg.groups, ... }: {
    isNormalUser = true;
    inherit extraGroups shell;
    openssh.authorizedKeys = {
      inherit keys;
    };
  };
in {
  options.xeserv.users = xeserv.users;
  
  config.users.users = {
    cadey = mkUser {
      keys = [ "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPg9gYKVglnO2HQodSJt4z4mNrUSUiyJQ7b+J798bwD9" ];
    };
    twi = mkUser {
      keys = [ "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPYr9hiLtDHgd6lZDgQMkJzvYeAXmePOrgFaWHAjJvNU" ];
    };
  };
}
```

[It's worth noting that `xeserv` in there can be anything you want. It's set to
`xeserv` as we are imagining that this is for the production environment of a
company named Xeserv.](conversation://Mara/hacker)

### Paranoid Settings

Next we're going to set up the paranoid settings from the last post into a
module named `paranoid.nix`. First we'll need to grab
[impermanence](https://github.com/nix-community/impermanence) into our niv
manifest like this:

```console
$ niv add nix-community/impermanence
Adding package impermanence
  Writing new sources file
Done: Adding package impermanence
```

Then open `common/default.nix` and change this line:

```nix
imports = [ ./users.nix ];
```

To something like this:

```nix
imports = [ ./paranoid.nix ./users.nix ];
```

Then open `./paranoid.nix` in a text editor and paste in the following:

```nix
# common/paranoid.nix

{ config, pkgs, lib, ... }:

with lib;

let
  sources = import ../nix/sources.nix;
  impermanence = sources.impermanence;
  cfg = config.xeserv.paranoid;
  
  ifNoexec = if cfg.noexec then [ "noexec" ] else [ ];
in {
  imports = [ "${impermanence}/nixos.nix" ];

  options.xeserv.paranoid = {
    enable = mkEnableOption "enables ephemeral filesystems and limited persistence";
    noexec = mkEnableOption "enables every mount on the system save /nix being marked as noexec (potentially dangerous at a social level)";
  };
  
  config = mkIf cfg.enable {
    fileSystems."/" = mkForce {
      device = "none";
      fsType = "tmpfs";
      options = [ "defaults" "size=2G" "mode=755" ] ++ ifNoexec;
    };
    
    fileSystems."/etc/nixos".options = ifNoexec;
    fileSystems."/srv".options = ifNoexec;
    fileSystems."/var/lib".options = ifNoexec;
    fileSystems."/var/log".options = ifNoexec;
    
    fileSystems."/boot" = {
      device = "/dev/disk/by-label/boot";
      fsType = "vfat";
    };

    fileSystems."/nix" = {
      device = "/dev/disk/by-label/nix";
      autoResize = true;
      fsType = "ext4";
    };

    boot.cleanTmpDir = true;

    environment.persistence."/nix/persist" = {
      directories = [
        "/etc/nixos" # nixos system config files, can be considered optional
        "/srv" # service data
        "/var/lib" # system service persistent data
        "/var/log" # the place that journald dumps it logs to
      ];
    };

    environment.etc."ssh/ssh_host_rsa_key".source =
      "/nix/persist/etc/ssh/ssh_host_rsa_key";
    environment.etc."ssh/ssh_host_rsa_key.pub".source =
      "/nix/persist/etc/ssh/ssh_host_rsa_key.pub";
    environment.etc."ssh/ssh_host_ed25519_key".source =
      "/nix/persist/etc/ssh/ssh_host_ed25519_key";
    environment.etc."ssh/ssh_host_ed25519_key.pub".source =
      "/nix/persist/etc/ssh/ssh_host_ed25519_key.pub";
    environment.etc."machine-id".source = "/nix/persist/etc/machine-id";
  };
}
```

This should give us the base that we need to build the system image for AWS.

## Building The Image

As I mentioned earlier we need to build a system image before we can build the
image. NixOS normally hides a lot of this magic from you, but we're going to
scrape away all that magic and do this by hand. In your `paranix-configs`
folder, create a folder named `images`. This creatively named folder is where we
will store our NixOS image generation scripts.

Copy this code into `build.nix`. This will tell NixOS to create a new system
closure with configuration in `images/configuration.nix`:

```nix
# images/build.nix

let
  sources = import ../nix/sources.nix;
  pkgs = import sources.nixpkgs { };
  sys = (import "${sources.nixpkgs}/nixos/lib/eval-config.nix" {
    system = "x86_64-linux";
    modules = [ ./configuration.nix ];
  });
in sys.config.system.build.toplevel
```

And in `images/configuration.nix` add this skeleton config:

```nix
# images/configuration.nix

{ config, pkgs, lib, modulesPath, ... }:

{
  imports = [ ../common (modulesPath + "/virtualisation/amazon-image.nix") ];
  
  xeserv.paranoid.enable = true;
}
```

[You can adapt this to other clouds by changing what module is imported. See the
list of available modules <a
href="https://github.com/NixOS/nixpkgs/tree/master/nixos/modules/virtualisation">here</a>.](conversation://Mara/hacker)

Then you can kick off the build with `nix-build`:

```console
$ nix-build build.nix
```

It will take a moment to assemble everything together and when you are done you
should have an entire functional system closure in `./result`:

```console
$ cat ./result/nixos-version
21.05pre-git
```

[It has `pre-git` here because we're using a pinned commit of the `nixos-21.05`
git branch. Release channels don't have that suffix there.](conversation://Mara/hacker)

From here we need to put this base system closure into a disk image for AWS.
This process is a bit more involved, but here are the high level things needed
to make a disk image for NixOS (or any Linux system for that matter):

- A virtual hard drive to install the OS to
- A partition mapping on the virtual hard drive
- Essential system files copied over
- A boot configuation

We can model this using a Nix function. This function would need to take in the
system config, some metadata about the kind of image to make and then it would
build the image and return the result. I've made this available
[here](https://tulpa.dev/cadey/paranix-configs/src/branch/main/images/make-image.nix)
so you can grab it into your config folder like this:

```console
$ wget -O make-image.nix https://tulpa.dev/cadey/paranix-configs/raw/branch/main/images/make-image.nix
```

Then we can edit `build.nix` to look like this:

```nix
# images/build.nix

let
  sources = import ../nix/sources.nix;
  pkgs = import sources.nixpkgs { };
  config = (import "${sources.nixpkgs}/nixos/lib/eval-config.nix" {
    system = "x86_64-linux";
    modules = [ ./configuration.nix ];
  });

in import ./make-image.nix {
  inherit (config) config pkgs;
  inherit (config.pkgs) lib;
  format = "vpc"; # change this for other clouds
}
```

Then you can build the AWS image with `nix-build`:

```console
$ nix-build build.nix
```

This will emit the AWS disk image in `./result`:

```console
$ ls ./result/
nixos.vhd
```

[AWS uses Microsoft Virtual PC hard disk files as the preferred input for their
vmimport service. This is probably a legacy thing.](conversation://Mara/hacker)

## Terraforming

[Terraform](https://www.terraform.io/) is not my favorite tool on the planet,
however it is quite useful for beating AWS and other clouds into shape. We will
be using Terraform to do the following:

- Create an S3 bucket to use for storing Terraform states in The Cloud
- Create an S3 bucket for the AMI base images
- Create an IAM role for importing AMIs
- Create an IAM role policy for allowing the AMI importer service to work
- Uploading the image to S3
- Import the image from S3 as an EBS snapshot
- Create an AMI from that EBS snapshot
- Create an example t2.micro virtual machine
- Deploy an example service config for nginx that does nothing

This sounds like a lot, but it's really not as much as it sounds. A lot of this
is boilerplate. The cost associated with these steps should be minimal.

In the root of your `paranix-configs` folder, make a folder called `terraform`,
as this is where our terraform configuration will live:

```console
$ mkdir terraform
$ cd terraform
```

Then you can proceed to the following steps.

### S3 State Bucket

In that folder, make a folder called `bootstrap`, this configuration will
contain the base S3 bucket config for Terraform state:

```console
$ mkdir bootstrap
$ cd bootstrap
```

Copy this terraform code into `main.tf`:

```hcl
# terraform/bootstrap/main.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "bucket" {
  bucket = "xeserv-tf-state-paranix"
  acl    = "private"

  tags = {
    Name = "Terraform State"
  }
}
```

Then run `terraform init` to set up the terraform environment:

```console
$ terraform init
```

It will download the AWS provider and run a few tests on your config to make
sure things are correct. Once this is done, you can run `terraform plan`:

```console
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions
are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.bucket will be created
  + resource "aws_s3_bucket" "bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = "private"
      + arn                         = (known after apply)
      + bucket                      = "xeserv-tf-state-paranoid"
      + bucket_domain_name          = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags                        = {
          + "Name" = "Terraform State"
        }
      + tags_all                    = {
          + "Name" = "Terraform State"
        }
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + versioning {
          + enabled    = (known after apply)
          + mfa_delete = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.
```

Terraform is very pedantic about what the state of the world is. In this case
nothing in the associated state already exists, so it is saying that it needs to
create the S3 bucket that we will use for our Terraform states in the future. We
can apply this with `terraform apply`:

```console
$ terraform apply
<the same thing as the plan>

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

If you want to perform these actions, follow the instructions.

```console
  Enter a value: yes
  
aws_s3_bucket.bucket: Creating...
aws_s3_bucket.bucket: Creation complete after 3s [id=xeserv-tf-state-paranoid]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Now that we have the state bucket, let's use it to create our AMI.

### Creating the AMI

In your `terraform` folder, create a new folder called `aws_image`. This is
where the terraform configuration for uploading our disk image to AWS will live.

```console
$ mkdir aws_image
$ cd aws_image
```

[This part of the config is modified from the instructions on how to create an
AMI from a locally created VM image <a
href="https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html">here</a>.](conversation://Mara/hacker)

Make a file called `main.tf` and we'll add to it as we go through this section.

In `main.tf`, add the following boilerplate to make the AWS provider use the
terraform state bucket we just created:

```hcl
# terraform/aws_image/main.tf

provider "aws" {
  region = "us-east-1"
}

terraform {
  backend "s3" {
    bucket = "xeserv-tf-state-paranoid"
    key    = "aws_image"
    region = "us-east-1"
  }
}
```

This will tell the AWS provider to use the S3 bucket we just made, but also to
put the terraform state in a key called `aws_image`. We will reuse this state
later for making our printer facts host. After we do this, we should run
`terraform init` to make sure that the state bucket is working:

```console
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v3.53.0...
- Installed hashicorp/aws v3.53.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Now let's create the S3 bucket that we will put our NixOS image in:

```hcl
# terraform/aws_image/main.tf

resource "aws_s3_bucket" "images" {
  bucket = "xeserv-ami-images"
  acl    = "private"

  tags = {
    Name = "Xeserv AMI Images"
  }
}
```

Then let's create the IAM role and policy that allows the VM importer service
to import objects from S3 into EBS snapshots that we use to create an AMI.

In the `aws_image` folder, copy this trust policy statement into
`vmie-trust-policy.json`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": { "Service": "vmie.amazonaws.com" },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals":{
                    "sts:Externalid": "vmimport"
                }
            }
        }
    ]
}
```

This will be used to give the VM import service permission to act against AWS on
your behalf.

In `main.tf`, add the following role and policy to the configuration:

```hcl
# terraform/aws_image/main.tf

resource "aws_iam_role" "vmimport" {
  name               = "vmimport"
  assume_role_policy = file("./vmie-trust-policy.json")
}

resource "aws_iam_role_policy" "vmimport_policy" {
  name   = "vmimport"
  role   = aws_iam_role.vmimport.id
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:GetBucketLocation"
      ],
      "Resource": [
        "${aws_s3_bucket.images.arn}",
        "${aws_s3_bucket.images.arn}/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:PutObject",
        "s3:GetBucketAcl"
      ],
      "Resource": [
        "${aws_s3_bucket.images.arn}",
        "${aws_s3_bucket.images.arn}/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:ModifySnapshotAttribute",
        "ec2:CopySnapshot",
        "ec2:RegisterImage",
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
    }
EOF
}
```

[Why do you define the trust policy in an external file but you have the role
policy defined inline?](conversation://Mara/hmm)

[Look at the `Resource`s defined in the `Statement` list. The S3 bucket in
question needs to be defined explicitly by its <a
href="https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html">ARN</a>,
and in order to give the vmimport service the minimal possible permissions, we
need to template out that policy JSON file, and doing this inline in Terraform
is a lot simpler.](conversation://Cadey/enby)

And now we should run `terraform plan` and `terraform apply` to make sure
everything works okay:

```console
$ terraform plan
<omitted>
Plan: 3 to add, 0 to change, 0 to destroy.

$ terraform apply
<omitted>
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

Perfect! Now we need to upload the image to S3. You are going to have to build
the NixOS image outside of terraform, so run `nix-build`:

```console
$ nix-build ../../build.nix
```

This should largely be a no-op and will put the correct `result` symlink in your
`aws_image` folder so terraform can read the image metadata.

[Practically you would want to make a script to run terraform, and in the script
for this folder you would probably want to add that `nix-build` command to that
script. However this is trivial and is thus an exercise for the
reader.](conversation://Mara/hacker)

In your `main.tf` file, add this:

```hcl
# terraform/aws_image/main.tf

resource "aws_s3_bucket_object" "nixos_21_05" {
  bucket = aws_s3_bucket.images.bucket
  key    = "nixos-21.05-paranoid.vhd"
  
  source = "./result/nixos.vhd"
  etag   = filemd5("./result/nixos.vhd")
}
```

Now we need to create the EBS snapshot. Copy this into your `main.tf`:

```hcl
# terraform/aws_image/main.tf

resource "aws_ebs_snapshot_import" "nixos_21_05" {
  disk_container {
    format = "VHD"
    user_bucket {
      s3_bucket = aws_s3_bucket.images.bucket
      s3_key    = aws_s3_bucket_object.nixos_21_05.key
    }
  }

  role_name = aws_iam_role.vmimport.name

  tags = {
    Name = "NixOS-21.05"
  }
}
```

This step may take a while (more than 5 minutes), so let's run `terraform plan`
and then `terraform apply`:

```console
$ terraform plan
Plan: 2 to add, 0 to change, 0 to destroy.

$ terraform apply
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Finally you can create the AMI and export the AMI ID like this:

```hcl
# terraform/aws_image/main.tf

resource "aws_ami" "nixos_21_05" {
  name                = "nixos_21_05"
  architecture        = "x86_64"
  virtualization_type = "hvm"
  root_device_name    = "/dev/xvda"
  ena_support         = true
  sriov_net_support   = "simple"

  ebs_block_device {
    device_name           = "/dev/xvda"
    snapshot_id           = aws_ebs_snapshot_import.nixos_21_05.id
    volume_size           = 40 # you can go as low as 8 GB, but 40 is a nice number
    delete_on_termination = true
    volume_type           = "gp3"
  }
}

output "nixos_21_05_ami" {
  value = aws_ami.nixos_21_05.id
}
```

Then run `terraform plan` and `terraform apply`:

```console
$ terraform plan
Plan: 1 to add, 0 to change, 0 to destroy.

$ terraform apply
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

nixos_21_05_ami = "ami-0f43f74cbbdd1ddef"
```

Et voila! We have a NixOS base image that we can use for production workloads.
Let's use it to create a NixOS server running the [printer facts
service](https://printerfacts.cetacean.club/).

[<big>**KEEP IN MIND**</big> that this configuration means that every time you
rebuild and upload this image you potentially risk breaking production machines.
Don't rebuild this config more than once every 6 months (or when you bump to a
new release of NixOS) at most.](conversation://Mara/hacker)

### Using the AMI

Let's make a new folder in the `terraform` folder called `printerfacts`. In this
folder we're going to set up a new terraform state that imports the AMI state we
just made and then we will use that AMI to run the printer facts service.

```console
$ mkdir printerfacts
$ cd printerfacts
```

In `main.tf`, copy the following:

```hcl
# terraform/printerfacts/main.tf

provider "aws" {
  region = "us-east-1"
}

terraform {
  backend "s3" {
    bucket = "xeserv-tf-state-paranoid"
    key    = "printerfacts"
    region = "us-east-1"
  }
}
```

Now you can `terraform init` as normal to ensure everything is working as we
expect:

```console
$ terraform init
Terraform has been successfully initialized!
```

Then let's add the `aws_image` state as a data source. This will let us
reference the AMI ID from the remote state file instead of having to build it
from scratch every time.

```hcl
# terraform/printerfacts/main.tf

data "terraform_remote_state" "aws_image" {
  backend = "s3"
  
  config = {
    bucket = "xeserv-tf-state-paranoid"
    key    = "aws_image"
    region = "us-east-1"
  }
}
```

AWS wants us to create a keypair for the instance, so to make AWS happy we will
make a keypair like this:

```hcl
# terraform/printerfacts/main.tf

resource "tls_private_key" "state_ssh_key" {
  algorithm = "RSA"
}

resource "aws_key_pair" "generated_key" {
  key_name   = "generated-key-${sha256(tls_private_key.state_ssh_key.public_key_openssh)}"
  public_key = tls_private_key.state_ssh_key.public_key_openssh
}
```

[You will need to `terraform init` after this step.](conversation://Mara/hacker)

Now we need to create a security group for this instance. This security group
should do the following:

- Allow port 22 (ssh) ingress
- Allow port 80 (http) ingress
- Allow ICMP (ping) ingress
- Allow ICMP (ping) egress
- Allow TCP egress on all ports to everywhere
- Allow UDP egress on all ports to everywhere

You can do this with this terraform fragment:

```hcl
# terraform/printerfacts/main.tf

resource "aws_security_group" "printerfacts" {
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 65535
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Then we can create the AWS instance using our AMI, keypair and security group:

```hcl
# terraform/printerfacts/main.tf

resource "aws_instance" "printerfacts" {
  ami           = data.terraform_remote_state.aws_image.outputs.nixos_21_05_ami
  instance_type = "t3.micro"
  security_groups = [
    aws_security_group.printerfacts.name,
  ]
  key_name = aws_key_pair.generated_key.key_name

  root_block_device {
    volume_size = 40 # GiB
  }

  tags = {
    Name = "xe-printerfacts"
  }
}
```

And then we can create a NixOS deploy config with the fantastic
[deploy_nixos](https://github.com/tweag/terraform-nixos/tree/master/deploy_nixos)
module from Tweag. Copy this into your `main.tf`:

```hcl
# terraform/printerfacts/main.tf

module "deploy_printerfacts" {
  source          = "git::https://github.com/Xe/terraform-nixos.git//deploy_nixos?ref=1b49f2c6b4e7537cca6dd6d7b530037ea81e8268"
  nixos_config    = "${path.module}/printerfacts.nix"
  hermetic        = true
  target_user     = "root"
  target_host     = aws_instance.printerfacts.public_ip
  ssh_private_key = tls_private_key.state_ssh_key.private_key_pem
  ssh_agent       = false
  build_on_target = false
}
```

[You will need to `terraform init` again after this
step.](conversation://Mara/hacker)

Now let's make the `printerfacts.nix` host definition. We're going to start with
a simple config to begin with. This will start nginx in a mostly broken but
still semi-functional state on port 80.

```nix
# terraform/printerfacts/printerfacts.nix

let
  sources = import ../../nix/sources.nix;
  pkgs = import sources.nixpkgs { };
  system = "x86_64-linux";

  configuration = { config, lib, pkgs, ... }: {
    imports = [
      ../../common
      "${sources.nixpkgs}/nixos/modules/virtualisation/amazon-image.nix"
    ];

    networking.firewall.allowedTCPPorts = [ 22 80 ];

    xeserv.paranoid.enable = true;

    services.nginx.enable = true;
  };
in import "${sources.nixpkgs}/nixos" { inherit system configuration; }
```

[What is up with that config? It doesn't look like a normal NixOS module at
all.](conversation://Mara/hmm)

[That is a NixOS config that will use the pinned version of nixpkgs with niv in
order to build everything. It won't work everywhere, however the `hermetic` flag
in the `deploy_nixos` Terraform module will make this
work.](conversation://Cadey/enby)

Now let's deploy all this and see if it works!

```console
$ terraform init

$ terraform plan

$ terraform apply
```

### Printerfacts Install

Now we can add the printerfacts service to the VM. First, add the printerfacts
repo to niv:

```console
$ niv add git -n printerfacts --repo https://tulpa.dev/cadey/printerfacts
Done: Adding package printerfacts
```

Then create a service definition for it in your `common` folder. First create
the folder `common/services`:

```
$ cd ../..
$ cd common
$ mkdir services
$ cd services
```

Then create a `default.nix` file with the following contents:

```nix
# common/services/default.nix

{ ... }:

{
  imports = [ ./printerfacts.nix ];
}
```

And create `./printerfacts.nix` with this service boilerplate:

```nix
# common/services/printerfacts.nix

{ config, pkgs, lib, ... }:

with lib;

let
  sources = import ../../nix/sources.nix;
  pkg = pkgs.callPackage sources.printerfacts { };
  cfg = config.xeserv.services.printerfacts;
in
{
  options.xeserv.services.printerfacts = {
    enable = mkEnableOption "enable Printerfacts";
    useACME = mkEnableOption "enable ACME certs";
    
    domain = mkOption {
      type = types.str;
      default = "printerfacts.akua";
      example = "printerfacts.cetacean.club";
      description =
        "The domain name that nginx should check against for HTTP hostnames";
    };
    
    port = mkOption {
      type = types.int;
      default = 28318;
      example = 9001;
      description =
        "The port number printerfacts should listen on for HTTP traffic";
    };
  };
  
  config = mkIf cfg.enable {
    systemd.services.printerfacts = {
      wantedBy = [ "multi-user.target" ];
      
      script = ''
        export PORT=${toString cfg.port}
        export DOMAIN=${toString cfg.domain}
        export RUST_LOG=info
        exec ${pkg}/bin/printerfacts
      '';

      serviceConfig = {
        Restart = "always";
        RestartSec = "30s";
        WorkingDirectory = "${pkg}";
        RuntimeDirectory = "printerfacts";
        RuntimeDirectoryMode = "0755";
        StateDirectory = "tailscale";
        StateDirectoryMode = "0750";
        CacheDirectory = "tailscale";
        CacheDirectoryMode = "0750";
        DynamicUser = "yes";
      };
    };
    
    services.nginx.virtualHosts."${cfg.domain}" = {
      locations."/" = {
        proxyPass = "http://127.0.0.1:${toString cfg.port}";
        proxyWebsockets = true;
      };
      enableACME = cfg.useACME;
    };
  };
}
```

Then wire up `common/default.nix` with this:

```nix
# common/default.nix

imports = [ ./paranoid.nix ./users.nix ./services ];
```

Then you can add this to your machine config in the terraform directory:

```nix
# terraform/printerfacts/printerfacts.nix

configuration = { config, lib, pkgs, ... }: {
  # ...
 
  xeserv.services.printerfacts = {
    enable = true;
    domain = "3.237.88.228"; # replace this with the IP of your AWS instance
  };
};
```

Then `terraform plan` and `terraform apply`:

```console
$ terraform plan

$ terraform apply
```

And finally get yourself a hard-earned printer fact:

```console
$ curl http://3.237.88.228/fact
In 1987 printers overtook scanners as the number one pet in America.
```

---

We have gone from nothing to a fully production-ready NixOS deployment including
a custom AMI pinned to an exact version of NixOS and an additional service added
from its git repo. This will allow you to create a NixOS deployment that can be
used by multiple people but will also stay pinned to an exact version of NixOS.
Terraform will do all of the NixOS building and ensure that things are kept up
to date, meaning that your infrastructure is all configured using the same
workflow.

This post outlines boilerplate and templates. I'm sure that you could easily
adapt these templates for other things as well. If you need to store persistent
data, make sure its being put in `/var/lib` so that it isn't wiped on reboot.
This took at least a week of research, banging my head against the wall and so
many failures to implement this. Many thanks to [Graham
Christensen](https://twitter.com/grhmc) for unblocking me on this and pulling me
back from the chasm a few times.

Hope this helps your prod NixOS adventures!
