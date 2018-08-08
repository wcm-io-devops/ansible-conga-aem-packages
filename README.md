# wcm_io_devops.conga-packages

This role provides a generic, reusable way to install all AEM packages generated for a [CONGA](http://devops.wcm.io/conga/) role to a running AEM instance. It is meant to be included or declared as dependency by other roles that need to install packages generated by CONGA. But it can also used directly if a CONGA role only generates AEM packages. In this case all that is needed to deploy the CONGA role to AEM is apply this role with the appropriate `conga_role_mapping` (see [wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts)).

## Requirements

This role requires Ansible 2.0 or higher. Maven must be installed since the role uses the [wcm.io Content Package Maven Plugin](http://wcm.io/tooling/maven/plugins/wcmio-content-package-maven-plugin/) for the actual package installation, 


## Role Variables

Available variables are listed below, along with their default values:

    conga_aem_packages_maven_host: "{{ conga_host | default('localhost') }}"

Host to execute Maven on

	conga_aem_packages_maven_cmd: mvn

Name of the Maven executable to use. 

    conga_aem_packages_maven_opts: "-B -U"

Maven options (run in batch mode, update snapshots)

    # conga_aem_packages_maven_settings: ~/.m2/settings.xml

Path of a custom settings file to use when running Maven

	conga_aem_packages_wcmio_content_package_maven_plugin_version: 1.6.0

The default version of the wcm.io content package maven plugin to use.
If a version is specified for `io.wcm.maven.plugins:wcmio-content-package-maven-plugin` in the `conga_version_info` fact (provided by [wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts)) this value will be overridden.

	conga_aem_packages_wcmio_content_package_maven_plugin_changed_output: "Package installed"

The string to looks for in the output of the package plugin to determine if a package was actually installed or skipped because it was already installed. This is important to avoid unnecessary lengthy restarts (e.g. when a service pack is part of package list).

	conga_aem_packages_port: "{{ conga_config.quickstart.port }}"

The port of the target AEM instance. It defaults to the port configured in the [`aem-cms`](https://github.com/wcm-io-devops/conga-aem-definitions/blob/develop/conga-aem-definitions/src/main/roles/aem-cms.yaml) role to make it automatically work for both author and publisher instances (provided that the package role inherits from this role so that it can access its configuration).

	conga_aem_packages_service_url: "http://{{ inventory_hostname }}:{{ conga_aem_packages_port }}/crx/packmgr/service"

The  package manager service URL the Maven plugin uses to poll the package state.

Additionally, the role expects the `conga_packages` variable to be set by the [wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts) role (on which this role depends) to the list of packages from the CONGA configuration model.

Enables/disables standalone mode. When set to true the
[wcm_io_devios.aem_service](https://github.com/wcm-io-devops/ansible-aem-service) dependency is
enabled. Set this value to false when you have several aem-service
dependencies in your play to avoid multiple AEM restarts.

Additionally, the role expects the `conga_packages` variable to be set
by the
[wcm_io_devios.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts) role
(on which this role depends) to the list of packages from the CONGA
configuration model.

## Dependencies

This role depends on the
[wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts) role
for supplying the list of packages to install. In the standalone mode it
also depends on the
[wcm_io_devops.aem_service](https://github.com/wcm-io-devops/ansible-aem-service) role
for ensuring that the target AEM instance is started and restarting it
if the package metadata declares that it is required.

## Example Playbook

This playbook sets the `conga_aem_packages_port`  variable from the CONGA configuration of the `aem-cms` role and then installs all packages generated by the role `my-application-role`.

	- hosts: aem
	  pre_tasks:
	    - conga_facts:
	        conga_role_mapping: aem-cms
	    - set_fact:
	        conga_aem_packages_port: "{{ conga_config.quickstart.port }}"
	
	  roles:
	    - { role: wcm_io_devops.conga_aem_packages, conga_role_mapping: my-application-role}

## License

Apache 2.0
