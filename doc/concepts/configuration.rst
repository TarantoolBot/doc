..  _configuration:

Configuration
=============

Tarantool provides the ability to configure a cluster by using a YAML configuration file.
A cluster's topology includes the following elements:

-   An *instance* is a member of a cluster that store data or act as a router for handling CRUD requests in a sharded cluster.
-   A *replica set* is a pack of instances that operate on copies of the same databases and provides redundancy and increases data availability.
-   A *group* provides the ability to organize replica sets. For example, in a sharded cluster one group can contain storage instances and another group can contain routers used to handle CRUD requests.

You can flexibly configure a cluster's settings on different levels: from global settings applied to all groups to parameters specific for concrete instances.

This topic describes main configuration approaches and gives an overview of configuration options available in Tarantool.




..  _configuration_approaches:

Configuration approaches
------------------------

Tarantool configuration parameters can be specified in different ways:

*   Configuration in a file
*   :ref:`Configuration in code <configuration_code>` (legacy)
*   Environment variables
*   Centralized configuration



..  _configuration_file:

Configuration in a file
~~~~~~~~~~~~~~~~~~~~~~~

.. _configuration_instance_basic:

Basic instance configuration
****************************

Single instance (``config.yaml``):

..  literalinclude:: /code_snippets/test/config/iproto_instance_scope.yaml
    :language: yaml
    :dedent:

To :ref:`run <configuration_run_instance>`:

.. code-block:: console

    tarantool --name instance-001 --config config.yaml


.. _configuration_scopes:

Configuration scopes
********************

Configuration scopes: instance, replicaset, group, and global.

-   **instance**: list instances, configure instance-specific settings

    ..  literalinclude:: /code_snippets/test/config/iproto_instance_scope.yaml
        :language: yaml
        :emphasize-lines: 6-9
        :dedent:

-   **replica set**: list replicasets (leader)

    ..  literalinclude:: /code_snippets/test/config/iproto_replicaset_scope.yaml
        :language: yaml
        :emphasize-lines: 4-7
        :dedent:

-   **group**: for example, storage or router (replication, application)

    ..  literalinclude:: /code_snippets/test/config/iproto_group_scope.yaml
        :language: yaml
        :emphasize-lines: 2-5
        :dedent:

-   **global**: applies to all instances in all groups (credentials, iproto, global sharding settings)

    ..  literalinclude:: /code_snippets/test/config/iproto_global_scope.yaml
        :language: yaml
        :emphasize-lines: 1-3
        :dedent:



.. _configuration_replica_set_scopes:

Replica set configuration and configuration scopes
**************************************************

Replica set configuration:

..  literalinclude:: /code_snippets/test/config/replicaset_manual.yaml
    :language: yaml
    :dedent:


Basic overview.

How the values are merged.

Option constraints: sections/options that can't appear in any scope
(mark such options in reference).

Links to Replication and Sharding tutorials.

Mention failover approaches.


.. _configuration_application:

Loading an application
**********************

``app`` section - module and custom app config (``config:get()``):

..  literalinclude:: /code_snippets/test/config/single_instance_app.yaml
    :language: yaml
    :dedent:

Lua file:

..  literalinclude:: /code_snippets/test/config/myapp.lua
    :language: lua
    :dedent:

Run output:

.. code-block:: console

    main/103/interactive/myapp I> Hello from app, instance-001!


Use case - different apps for different groups:

.. code-block:: yaml

    groups:
      storages:
        app:
          module: storage
          # ...
      routers:
        app:
          module: router
          # ...

See :ref:`Launching a binary program <app_server-launching_app_binary>`.



.. _configuration_templating:

Templating
**********

Replacing variables with actual values at runtime. Example:

.. code-block:: yaml

    # ...
    instance-001:
      snapshot:
        dir: ./var/{{ instance_name }}/snapshots
      wal:
        dir: ./var/{{ instance_name }}/wals




..  _configuration_environment_variable:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

Starting from version :doc:`2.8.1 </release/2.8.1>`, you can specify configuration parameters via special environment variables.
The name of a variable should have the following pattern: ``TT_<NAME>``,
where ``<NAME>`` is the uppercase name of the corresponding :ref:`box.cfg parameter <box-cfg-params-ref>`.

For example:

* ``TT_LISTEN`` -- corresponds to the ``box.cfg.listen`` option.
* ``TT_MEMTX_DIR`` -- corresponds to the ``box.cfg.memtx_dir`` option.

In case of an array value, separate the array elements by comma without space:

..  code-block:: console

    export TT_REPLICATION="localhost:3301,localhost:3302"

If you need to pass :ref:`additional parameters for URI <index-uri-several-params>`, use the ``?`` and ``&`` delimiters:

..  code-block:: console

    export TT_LISTEN="localhost:3301?param1=value1&param2=value2"

An empty variable (``TT_LISTEN=``) has the same effect as an unset one meaning that the corresponding configuration parameter won't be set when calling ``box.cfg{}``.


.. _configuration_centralized:

Centralized configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

..  admonition:: Enterprise Edition
    :class: fact

    Centralized configuration is supported by the `Enterprise Edition <https://www.tarantool.io/compare/>`_ only.

Local config:

..  literalinclude:: /code_snippets/test/config/etcd.yaml
    :language: yaml
    :dedent:

Put a remote config:

.. code-block:: console

    etcdctl put /example/config/all.yaml < remote_config.yaml

Run:

.. code-block:: console

    tarantool -n instance-001 -c local_config.yaml
    tarantool -n instance-002 -c local_config.yaml
    tarantool -n instance-003 -c local_config.yaml


Describe how to reload configuration.


..  _configuration_precedence:

Configuration precedence
~~~~~~~~~~~~~~~~~~~~~~~~

-   `TT_*`
-   config
-   etcd
-   `TT_*_DEFAULT`
-   :ref:`tt configuration <tt-config>`



..  _configuration_options_overview:

Configuration options overview
------------------------------

.. _configuration_options_connection:

Connection settings
~~~~~~~~~~~~~~~~~~~

Port:

.. code-block:: yaml

    iproto:
      listen:
        "3301"

Address:

.. code-block:: yaml

    iproto:
      listen:
        "127.0.0.1:3301"


Several:

.. code-block:: yaml

    iproto:
      listen:
        "127.0.0.1:3301, 127.0.0.1:3303"


Parameters:

.. code-block:: yaml

    iproto:
      listen:
        "127.0.0.1:3301?p1=value1&p2=value2"

..  TODO
    SSL settings: localhost:3301?transport=ssl&ssl_key_file=<...>


Unix socket:

.. code-block:: yaml

    iproto:
      listen: 'unix/:./{{ instance_name }}.iproto'


.. _configuration_options_access_control:

Access control
~~~~~~~~~~~~~~

..  literalinclude:: /code_snippets/test/config/replicaset_manual.yaml
    :language: yaml
    :lines: 1-8
    :dedent:

See also: replication, access control.

.. _configuration_options_memory:

Memory
~~~~~~

``memtx.memory``:

.. code-block:: yaml

    memtx:
      memory: 100000000


.. _configuration_options_directories:

Snapshots and write-ahead logs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Snapshot and WAL directories:

.. code-block:: yaml

    instance-001:
      snapshot:
        dir: 'var/snapshots'
      wal:
        dir: 'var/wals'



.. _configuration_options_traffic_encryption:

Traffic encryption
~~~~~~~~~~~~~~~~~~

..  admonition:: Enterprise Edition
    :class: fact

    Traffic encryption is supported by the `Enterprise Edition <https://www.tarantool.io/compare/>`_ only.

Example: TODO

See also: :ref:`Traffic encryption <enterprise-iproto-encryption>`.




.. _configuration_run_instance:

Running a Tarantool instance
----------------------------

Command:

..  code-block:: console

    $ tarantool [OPTION ...]

Tarantool is started by entering either of the following command:

*   3.0 and higher:

    ..  code-block:: console

        $ tarantool --name INSTANCE_NAME --config CONFIG_FILE_PATH [OPTION ...]

*   2.11 and earlier:

    ..  code-block:: console

        $ tarantool INITIALIZATION_FILE_PATH [OPTION ...]



.. _configuration_command_options:

Command-line options
~~~~~~~~~~~~~~~~~~~~

Options that can be passed when :ref:`running a Tarantool instance <configuration_run_instance>`:

..  option:: -h, --help

    Print an annotated list of all available options and exit.

.. _index-tarantool_version:

..  option:: -v, -V, --version

    Print the product name and version.

    **Example**

    ..  code-block:: console

        % tarantool --version
        Tarantool 3.0.0-entrypoint-746-g36ef3fb43
        Target: Darwin-arm64-Release
        ...

    In this example:

    *   ``3.0.0`` is a Tarantool version.
        Tarantool follows semantic versioning, which is described in the :ref:`Tarantool release policy <release-policy>` section.

    *   ``Target`` is the platform Tarantool is built on.
        Platform-specific details may follow this line.


..  option:: -e EXPR

    Execute the 'EXPR' string. See also: `lua man page <https://www.lua.org/manual/5.3/lua.html>`_.

    **Example**

    ..  code-block:: console

        % tarantool -e "print('Hello, world!')"
        Hello, world!

..  option:: -l NAME

    Require the 'NAME' library. See also: `lua man page <https://www.lua.org/manual/5.3/lua.html>`_.

    **Example**

    ..  code-block:: console

        % tarantool -l luatest.coverage script.lua

..  option:: -j cmd

    Perform a LuaJIT control command. See also: `Command Line Options <https://luajit.org/running.html>`_.

    **Example**

    ..  code-block:: console

        % tarantool -j off app.lua

..  option:: -b ...

    Save or list bytecode. See also: `Command Line Options <https://luajit.org/running.html>`_.

    **Example**

    ..  code-block:: console

        % tarantool -b test.lua test.out

..  option:: -d SCRIPT

    Activate a debugging session for 'SCRIPT'. See also: `luadebug.lua <https://github.com/tarantool/tarantool/blob/master/third_party/lua/README-luadebug.md>`_.

    **Example**

    ..  code-block:: console

        % tarantool -d app.lua


..  option:: -i [SCRIPT]

    Enter an :ref:`interactive mode <interactive_console>` after executing 'SCRIPT'.

    **Example**

    ..  code-block:: console

        % tarantool -i


..  option:: --

    Stop handling options. See also: `lua man page <https://www.lua.org/manual/5.3/lua.html>`_.


..  option:: -

    Stop handling options and execute the standard input as a file. See also: `lua man page <https://www.lua.org/manual/5.3/lua.html>`_.




..  toctree::
    :hidden:

    configuration/configuration_code
    configuration/configuration_migrating
