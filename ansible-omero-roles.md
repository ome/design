# OMERO component Ansible roles

This is a proposal for replacing the existing [omero-server Ansible role](https://github.com/openmicroscopy/ansible-role-omero-server) with a set of omero component roles to allow the installation of decoupled OMERO components, initially server and web.


## Rationale

The existing [omero-server Ansible role](https://github.com/openmicroscopy/ansible-role-omero-server) handles most of the steps for setting up OMERO.server, with the exception of dependencies which are installed by other roles.
This includes installing and configuring both OMERO.server and OMERO.web, and although OMERO.web can be disabled it is not possible to install OMERO.web only.

Further more this role bundles `omego` which could potentially be used independently of this role.
In addition Ansible 2.2 has a bug which means `omego` is installed into the wrong virtualenv, breaking the role.


## Proposal

Split `omero-server` into multiple Ansible roles:

### omero-common
- Create an `omero` system user
- Create `/opt/omero` as the parent directory for all OMERO components
- Possible: Include `omero` and `omero-web` restart handlers so other roles can restart the server or web.

### omero-omego
- Install omego into a virtualenv: `/opt/omero/omego`
- Alternative: install `omego` globally and name this role `omego`

### omero-web
- Parent directory: `/opt/omero/web`
- Possible: Use a virtualenv with system-site-packages for OMERO.web: `/opt/omero/web/venv`, as an alternative to installing Django and gunicorn globally
- Download `OMERO.py`, symlink to `/opt/omero/web/OMERO.web`

### omero-server
- Refactor the existing role
- Parent directory: `/opt/omero/server`
- Download `OMERO.server`, symlink to `/opt/omero/server/OMERO.server`
- Web not supported


## Changes required to external components

### omego
- Ideally `omego` should be able to [symlink a downloaded package](https://github.com/ome/omego/pull/97).
- Possible: `omego upgrade` could upgrade `OMERO.web`, i.e. copy the configuration. An alternative is to manage all configuration in an [Ansible friendly manner outside OMERO](https://github.com/openmicroscopy/design/issues/70).

### Existing playbooks
- Existing playbooks will have to be updated if they don't use a tagged version of `omero-server`


## Main benefits

- OMERO components can be managed separately, supporting the future aim of decoupling OMERO.web.
- OMERO.web and OMERO.server configurations can be managed independently. For example a configuration that only affects web can be made without restarting OMERO.server.
