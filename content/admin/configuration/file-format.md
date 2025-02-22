---
title: File Format
toc: true
menu: File Format
order: 10
---

# Yaml File Format

The configuration file will be loaded the first time you start
`ejabberd`. The configuration file name MUST have “.yml” or “.yaml”
extension. This helps ejabberd to differentiate between the new and
legacy file formats (see
section [Legacy Configuration File](#legacy-configuration-file)).

The configuration file is written in [`YAML`](https://en.wikipedia.org/wiki/YAML).

Note that `ejabberd` never edits the configuration file. If you are
changing parameter from web admin interface, you will need to apply
them to configuration file manually. This is to prevent messing up
with your config file comments, syntax, etc.

Please, consult `ejabberd.log` for configuration errors. `ejabberd` will
report syntax related errors, as well as complains about unknown options
and invalid values. Make sure you respect indentation (YAML is
sensitive to this) or you will get pretty cryptic errors.

# Legacy Configuration File

In previous `ejabberd` version the configuration file should be
written in Erlang terms. The format is still supported, but it is
highly recommended to convert it to the new YAML format with the
[convert_to_yaml](/developer/ejabberd-api/admin-api/#convert-to-yaml)
API command using [ejabberdctl](/admin/guide/managing/#ejabberdctl).

If you want to specify some options using the old Erlang format, you
can set them in an additional cfg file, and include it using the
`include_config_file` option, see
[Include Additional Configuration Files](#include-additional-configuration-files)
for the option description and a related example in
[Restrict Execution with AccessCommands](/admin/guide/managing/#restrict-execution-with-accesscommands).


# Include Additional Files

The option [include_config_file](/admin/configuration/toplevel/#include-config-file)
 in a configuration file instructs
`ejabberd` to include other configuration files immediately.

This is a basic example:

	include_config_file: /etc/ejabberd/additional.yml

In this example, the included file is not allowed to contain a `listen`
option. If such an option is present, the option will not be accepted.
The file is in a subdirectory from where the main configuration file is.

	include_config_file:
	  ./example.org/additional_not_listen.yml:
	    disallow: [listen]

Please notice that options already defined in the main configuration file
cannot be redefined in the included configuration files.
But you can use [host_config](/admin/configuration/toplevel/#host-config)
and [append_host_config](/admin/configuration/toplevel/#append-host-config)
as usual (see [Virtual Hosting](/admin/configuration/basic/#virtual-hosting)).

In this example, `ejabberd.yml` defines some ACL for the whole ejabberd server, and later includes another file:

	acl:
	  admin:
	    user:
	      - admin@localhost
	include_config_file:
	  /etc/ejabberd/acl.yml

The file `acl.yml` can add additional administrators to one of the virtual hosts:

	append_host_config:
	  localhost:
	    acl:
	      admin:
	        user:
	          - bob@localhost
	          - jan@localhost

# Macros in Configuration File

In the `ejabberd` configuration file, it is possible to define a macro
for a value and later use this macro when defining an option.

A macro is defined using the [define_macro](/admin/configuration/toplevel/#define-macro) option.

This example shows the basic usage of a macro:


	define_macro:
	  LOG_LEVEL_NUMBER: 5
	loglevel: LOG_LEVEL_NUMBER

The resulting option interpreted by `ejabberd` is: `loglevel: 5`.

This example shows that values can be any arbitrary YAML value:


	define_macro:
	  USERBOB:
	    user:
	      - bob@localhost
	acl:
	  admin: USERBOB

The resulting option interpreted by `ejabberd` is:


	acl:
	  admin:
	    user:
	      - bob@localhost

This complex example:


	define_macro:
	  NUMBER_PORT_C2S: 5222
	  NUMBER_PORT_HTTP: 5280
	listen:
	  -
	    port: NUMBER_PORT_C2S
	    module: ejabberd_c2s
	  -
	    port: NUMBER_PORT_HTTP
	    module: ejabberd_http

produces this result after being interpreted:


	listen:
	  -
	    port: 5222
	    module: ejabberd_c2s
	  -
	    port: 5280
	    module: ejabberd_http

