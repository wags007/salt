======================
Command Line Reference
======================

Salt can be controlled by a command line client by the root user on the Salt 
master. The Salt command line client uses the Salt client API to communicate
with the Salt master server. The Salt client is straightforward and simple 
to use.

Using the Salt client commands can be easily sent to the minions.

Each of these commands accepts an explicit `--config` option to point to either
the master or minion configuration file.  If this option is not provided and
the default configuration file does not exist then Salt falls back to use the
environment variables ``SALT_MASTER_CONFIG`` and ``SALT_MINION_CONFIG``.

.. seealso::

    :doc:`/topics/configuration`

Using the Salt Command
======================

The Salt command needs a few components to send information to the salt
minions. The target minions need to be defined, the function to call and any
arguments the function requires.

Defining the Target Minions
---------------------------

The first argument passed to salt, defines the target minions, the target
minions are accessed via their hostname. The default target type is a bash
glob:

.. code-block:: bash

    salt '*foo.com' sys.doc


Salt can also define the target minions with regular expressions:

.. code-block:: bash

    salt -E '.*' cmd.run 'ls -l | grep foo'

Or to explicitly list hosts, salt can take a list:

.. code-block:: bash

    salt -L foo.bar.baz,quo.qux cmd.run 'ps aux | grep foo'

More Powerful Targets
---------------------

The simple target specifications, glob, regex and list will cover many use
cases, and for some will cover all use cases, but more powerful options exist.

Targeting with Grains
`````````````````````

The Grains interface was built into Salt to allow minions to be targeted by
system properties. So minions running on a particular operating system can
be called to execute a function, or a specific kernel.

Calling via a grain is done by passing the -G option to salt, specifying
a grain and a regular expression to match the value of the grain.

.. code-block:: bash

    salt -G 'os:Fedora' test.ping

Will return True from all of the minions running Fedora.

To discover what grains are available and what the values are, execute the
grains.item salt function:

.. code-block:: bash

    salt '*' grains.items

Targeting with Executions
`````````````````````````

As of 0.8.8 targeting with executions is still under heavy development and this
documentation is written to refernce the behavior of execution matching in the
future.

Execution matching allows for a primary function to be executed, and then based
on the return of the primary function the main function is executed.

Execution matching allows for matching minions based on any arbitrairy running
data on tne minions.

Compound Targeting
``````````````````
Multiple target interfaces can be used in conjunction to determine the command
targets. These targets can then be combined using and or or statements. This
is well defined with an example:

.. code-block:: bash

    salt -C 'G@os:Debian and webser* or E@db.*' test.ping

in this example any minion who's id starts with webser and is running Debian,
or any minion who's id starts with db will be matched.

The type of matcher defaults to glob, but can be specified with the
corresponding letter followed by the @ symbol. In the above example a grain is
used with G@ as well as a regular expression with E@. The webser* target does
not need to be prefaced with a target type specifier because it is a glob.

Calling the Function
--------------------

The function to call on the specified target is placed after the target
specification.

Finding available minion functions
``````````````````````````````````

The Salt functions are self documenting, all of the function documentation can
be retried from the minions via the :func:`sys.doc` function:

.. code-block:: bash

    salt '*' sys.doc

Compound Command Execution
--------------------------

If a series of commands need to be sent to a single target specification then
the multiple commands can be send in a single publish. This can make gathering
groups of information faster, and lowers the stress on the network for repeated
commands.

Compound command execution works by sending a list of functions and arguments
instead of sending a single function and argument. The functions are executed
on the minion in the order they are defined on the command line, and then the
data from all of the commands are returned in a dictionary. This means that
the set of commands are called in a predictable way, and the returned data can
be easily interpreted.

Executing compound commands if done by passing a comma delimited list of
functions, followed by a comma delimited list of arguments:

.. code-block:: bash

    salt '*' cmd.run,test.ping,test.echo 'cat /proc/cpuinfo',,foo

The trick to look out for here, is that if a function is being passed no
arguments, then there needs to be a placeholder for the absent arguments. This
is why in the above example, there are two commas right next to each other.
``test.ping`` takes no arguments, so we need to add another comma, otherwise
Salt would attempt to pass "foo" to ``test.ping``.
