# git-deployable-app
[![Build Status](https://semaphoreci.com/api/v1/rubyisbeautiful/git-deployable-app/branches/master/badge.svg)](https://semaphoreci.com/rubyisbeautiful/git-deployable-app)
[![Build Status](https://travis-ci.org/rubyisbeautiful/git-deployable-app.svg?branch=master)](https://travis-ci.org/rubyisbeautiful/git-deployable-app)


This role can deploy git based deployments

It is simple (even stupid), opinionated, but flexible.  The ideal environment
is one in which the Ansible controller is ephemeral (e.g. container-based),
and the defaults support this.  Upon running a playbook with this role, a git
repo will be shallowly cloned **on the controller** for each configured app,
archived, copied to remote targets, and unarchived.

It will mindlessly deploy to the same target dir, overwriting whatever was there
previously and leaving in place things that might have been removed (if you
are using static file systems or servers, e.g.).  As is, this is
not a good replacement for something like Capistrano
(see
[Ansistrano](https://galaxy.ansible.com/carlosbuenosvinos/ansistrano-deploy/)
You could achieve a simple similar effect by using a changing
`gda_base_remote_path` for example.


Requirements
------------

The controller requires Git 2+ to make use of the depth argument


Role Variables
--------------

You must set the following variables, either in data, a playbook or on the
command line:

`gda_apps` is a hash of apps to deploy.

Example Data:

```
gda_apps:
  foo:
    url: ssh://git@github.com/whatever/foo.git
    path: /apps/foo
```

**NB** Due to an ansible restriction, you will not want to use `-` in the keys
under `gda_apps`.  This can apparently be overriden with an ansible strict
setting but I haven't tried.

#### Required Keys:

`gda_apps`.`url` - the git url of the app

`gda_apps`.``path` - the path of the resulting app

#### Optional Keys:

`gda_apps`.`subdir` - if given, only this subdir will be archived and deployed

`gda_apps`.`owner` - if given, the owner of the resulting deployed app

`gda_apps`.`group` - if given, the group of the resulting deployed app

`gda_apps`.`version` - if given, the version to check out (branch, tag, SHA)

`gda_apps`.`repo_local_path` - if given, the path to checkout the repo on the
                              controller

`gda_apps`.`force` - if given, force the repo to checkout/update


### Defaults

`gda_base_local_path`
The location to checkout repos.  Default is `/tmp`

`gda_base_remote_path`
The base remote path for apps.  This is only used when an app path is relative.
Default is `/apps`

`gda_repo_version`
The default version to checkout, if not supplied per app, is `master`

`gda_repo_force`
By default, this is `false`

`gda_repo_depth`
The default history depth is `1` by default

`gda_repo_clone`
This is `true` and if you change it it false, will result in basically
nothing happening for that app.

`gda_archive_format`
Possible choices are `gz` (default), `bz2`, `zip`

`gda_owner`
default is `nobody`. 

`gda_group`
default is `nogroup`

`gda_archive_mode`
default is `0644` and is passed directly to Ansible's archive module

`gda_dir_mode`
default is 0755 works together with `gda_fix_dir_permissions`

`gda_fix_dir_permissions`
default is true. because the local / controller will not necessarily have the
users as the remote / target nodes, the ownership has to be set post hoc.  And
in many cases, for some reasons probably related to the same cause, the mode
of the directories as well.  Leave this to true to perform the post deploy fix,
set to false to disable it.  A future version will attempt fixes on files,
which needs to be more nuanced perhaps.  You can of course do any / change any
of this in post tasks.


Dependencies
------------

N/A

Example Playbooks
----------------


#### Basic

```
- hosts: all
  roles:
    - git-deployable-app
```

License
-------

MIT


Author Information
------------------

rubyisbeautiful
