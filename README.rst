=========================
Oríon Tutorial & Hackaton
=========================


------------
Introduction
------------

Why should you care... 
======================
(As Brady would say 😊)

... about Hyper-parameter optimization
--------------------------------------

Hyper-parameters are important. Researchers cannot make sound benchmarks without proper
hyper-parameter optimization. We are all human, we love our research, we cannot be expected to spend
as much time optimizing hyper-parameters of our benchmarks as much as we do on our beloved models.
**We need an automated tool which takes care of it impartially**.

Hyper-parameters can be difficult to optimize. If your idea works, you don't want to fail just
because you can't find the right hyper-parameter values. **We need an automated tool which takes
care of it more efficiently and more accurately**.

... about Oríon
---------------

It is focusing on **usability** and provides an interface which is **intuitive** as well as
**flexible** enough to support many different workflows.

It provides a **platform** for black-box optimization algorithms with the goal of building a
**community** connecting together users of hyper-parameter optimization tools to developers of such
algorithms.

It is **open-source**.


It's not a journaling system
============================

We prefer to build tools with a lean design and specific goals. Oríon is not about saving data about
your experiments. It saves the minimal amount necessary to do hyper-parameter optimization, that is,
a single objective value.

If you are looking for a tool to do automatic journaling of your experiments then you may want to
take a look at the one we are developing, kleio_. Note that it is a prototype and do not
follow a stable development with backward compatibility like Oríon.

Where to find us
================

Do not hesitate to ping us on slack_, open issues, make feature requests or pull requests on github_
or even start discussions on our Discourse_.

The team and emails can be found on our ICML 2018 RML workshop paper_.

.. _paper: https://openreview.net/forum?id=r1xkNLPixX

-----
Setup
-----

Installation
============

Oríon supports python 3.5, 3.6 and 3.7. Please don't run away if you still use python2.7, we can
help you find redemption during this hackaton. 😊

On Mila computers, you can create a conda virtual environment with python 3.6.

.. code-block:: bash

    $ conda create -n orion.tutorial python=3.6
    ...
    Proceed ([y]/n)? y
    ...
    $ source activate orion.tutorial

Make sure you have the right ``pip``. It should be under ``$HOME/.conda`` by
default.
   
.. code-block:: bash

    (orion.tutorial) [SLURM] bouthilx@leto05:~$ which pip
    /u/bouthilx/.conda/envs/orion.tutorial/bin/pip

.. .. code-block:: bash
.. 
..     $ pip3 install --user virtualenvwrapper
.. 
.. Add this at the end of file ``~/.bashrc``.
.. 
.. .. code-block:: bash
..     
..     source $HOME/.local/bin/virtualenvwrapper.sh
..     export WORKON=$HOME/.virtualenvs
.. 
.. .. code-block:: bash
.. 
..     $ mkvirtualenv --python /usr/bin/python3.5 orion.tutorial


Core of Oríon
-------------

Installing Oríon is as simple as

.. code-block:: bash

    $ pip install orion.core

Plugins
-------

You can install plugins to add other algorithms or database backends to Oríon. For now the only
plugin supported is a Bayesian Optimizer algorithm wrapped from scikit-optimize.

.. code-block:: bash

    $ pip install orion.algo.skopt


Database configuration
======================

Each system you use (Mila, Cedar, Graham) should have their own configuration file.

Unix:

``$HOME/.config/orion.core/orion_config.yaml``

OSX:

``$HOME/Library/Application Support/orion.core/orion_config.yaml``

If not sure, you can run orion in verbose mode to see where it is looking for
configuration files. Here is an example of the output on Ubuntu

.. code-block:: bash

    $ orion -vv --debug hunt -n dummy
    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/home/bouthilx/.virtualenvs3.6/orion.list/share/orion.core/orion_config.yaml.example'
    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/etc/xdg/xdg-ubuntu/orion.core/orion_config.yaml'
    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/home/bouthilx/.virtualenvs3.6/orion.list/share/orion.core/orion_config.yaml.example'
    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/etc/xdg/xdg-ubuntu/orion.core/orion_config.yaml'
    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/home/bouthilx/.config/orion.core/orion_config.yaml'


Once you selected your file path for the database configuration, you simply
need to copy the following snippet and adapt the corresponding fields:

+--------------------+----------------------------------------------------------------------------------------------------------------+
| <USERNAME>         | Same as your username on Mila computers                                                                        |
+--------------------+----------------------------------------------------------------------------------------------------------------+
| <PASSWORD>         | Your password for the database, not the one on Mila computers                                                  |
+--------------------+----------------------------------------------------------------------------------------------------------------+
| <DATABASENAME>     | By default we assign database names which are your usernames.                                                  |
+--------------------+----------------------------------------------------------------------------------------------------------------+
| <CERTIFICATE-PATH> | On Mila computers it is ``~lisa/mongo.pem``.                                                                   |
|                    | If you use Oríon on other computers, you need to copy the certificate and adapt the configuration accordingly. |
+--------------------+----------------------------------------------------------------------------------------------------------------+

.. code-block:: yaml

    database:
      type: 'mongodb'
      name: <DATABASENAME>
      host: 'mongodb://<USERNAME>:<PASSWORD>@mongomila.iro.umontreal.ca:27017/<DATABASENAME>?ssl=true&ssl_ca_certs=<CERTIFICATE-PATH>&authSource=<DATABASENAME>'

Test connection
---------------

You can first check that everything works as expected by testing with the
``debug`` mode. This mode bypass the database in the configuration. If you run
the following command, you should get the following error.

.. code-block:: bash

    $ orion --debug hunt -n dummy
    ...
    AttributeError: 'str' object has no attribute 'configuration'

That's a terrible error message. -_- Note to ourselves; Improve this error message. What this should
tell is that the connection to database was successful but Oríon could not find any script to
optimize.

Now remove the option ``--debug`` to test the database. If it fails to connect,
you will get the following error. Otherwise, you'll get the (terrible) error above again
if it succeeded. Note that a connection failure will hang for approximately 60
seconds before giving up. ☕ time.

.. code-block:: bash

    $ orion hunt -n dummy
    ...
    orion.core.io.database.DatabaseError: Connection Failure: database not found on specified uri

If it fails, try running with ``-vv`` and make sure your configuration file is
properly found. Suppose your file path is ``/u/user/.config/orion.config/orion_config.yaml``, then you should **NOT**
see the following line in the output otherwise it means it is not found.

.. code-block:: bash

    DEBUG:orion.core.io.resolve_config:[Errno 2] No such file or directory: '/u/user/.config/orion.config/orion_config.yaml'

When you are sure the configuration file is found, look for the configuration
used by Oríon to initiate the DB connection.

.. code-block:: bash

    DEBUG:orion.core.io.experiment_builder:Creating mongodb database client with args: {'name': 'bouthilx', 'host': 'mongodb://bouthilx:<PASSWORD>@mongomila.iro.umontreal.ca:27017/bouthilx?ssl=true&ssl_ca_certs=/u/lisa/mongo.pem&authSource=bouthilx'}

Make sure you have the proper database name, database type and host URI.

-----
Usage
-----

To help understand how to use Oríon, this tutorial will walk you through each
steps to adapt a script from ``pytorch/examples.git``. First to install

.. code-block:: bash

    $ source activate orion.tutorial
    $ pip install torch torchvision
    $ git clone git@github.com:pytorch/examples.git

Adapting the script
===================

There is a few modifications needed to be able to use the script with Oríon.
First we need to make it executable.

.. code-block:: bash

    $ cd examples/mnist
    $ chmod +x main.py

Then we need to add a shebang at the top of ``main.py``. Note: We could drop those requirements in the
future.

.. code-block:: python

    #!/usr/bin/env python

At the top of the file, below the imports, import the helper function
``orion.client.report_results()``

.. code-block:: python

    from orion.client import report_results

We are almost done now. We need to add a line to the function ``test()`` so
that it returns the error rate.

.. code-block:: python

    return 1 - (correct / len(test_loader.dataset))

And finally, we get back this test error rate and call ``report_results`` to
return the objective value to Oríon. Note that ``report_results`` is meant to
be called only once, this is because Oríon only optimizes looking at 1
objective value. 

.. code-block:: python

        test_error_rate = test(args, model, device, test_loader)

    report_results([dict(
        name='test_error_rate',
        type='objective',
        value=test_error_rate)])

You can also return result types of ``'gradient'`` and ``'constraint'`` for
algorithms which supports those results as well.

Important note here, we use test error rate for sake of simplicity, because the
script does not contain validation dataset loader as-is, but we should
**never** optimize our hyper-parameters on the test set. We should always use a
validation set.

Another important note, Oríon will always **minimize** the objective so make sure you never try to
optimize something like the accuracy of the model unless you are looking for very very bad models.

Execution
=========

Once the script is adapted, optimizing the hyper-parameters with Oríon is
rather simple. Normally you would call the script the following way.

.. code-block:: bash

    $ ./main.py --lr 0.01

To use it with Oríon, you simply need to prepend the call with
``orion hunt -n <some name>`` and specify the hyper-parameter prior
distributions.

.. code-block:: bash

    $ orion hunt -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)'

This commandline call will sequentially execute ``./main.py --lr=<value>`` with random
values sampled from the distribution ``loguniform(1e-5, 1.0)``. We support all
distributions from scipy.stats_, plus ``choices()`` for categorical
hyper-parameters (similar to numpy's `choice function`_).

.. _scipy.stats: https://docs.scipy.org/doc/scipy/reference/stats.html
.. _`choice function`: https://docs.scipy.org/doc/numpy/reference/generated/numpy.random.choice.html

Experiments are interruptible, meaning that you can stop them either with
``<ctrl-c>`` or with kill signals. If your script is not resumable automatically then resuming an
experiment will restart your script from scratch.

You can resume experiments using the same commandline or simply by specifying
the name of the experiment.

.. code-block:: bash

    $ orion hunt -n orion-tutorial

Note that experiment names are unique, you cannot create two different
experiment with the same name.

You can also register experiments without executing them.

.. code-block:: bash

    $ orion init_only -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)'

Debugging
---------

When preparing a script for hyper-parameter optimization, we recommend first testing with ``debug``
mode. This will use an in-memory database which will be flushed at the end of execution. If you
don't use ``--debug`` you will likely quickly fill your database with broken experiments.

.. code-block:: bash

    $ orion --debug hunt -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)'

Hunting Options
---------------

.. code-block:: bash

    $ orion hunt --help
    
    Oríon arguments (optional):
      These arguments determine orion's behaviour
    
      -n stringID, --name stringID
                            experiment's unique name; (default: None - specified
                            either here or in a config)
      -c path-to-config, --config path-to-config
                            user provided orion configuration file
      --max-trials #        number of jobs/trials to be completed (default:
                            inf/until preempted)
      --pool-size #         number of concurrent workers to evaluate candidate
                            samples (default: 10)

``name``

The unique name of the experiment.

``config``

Configuration file for Oríon which may define the database, the algorithm and all options of the
command hunt, including ``name``, ``pool-size`` and ``max-trials``.

``max-trials``

The maximum number of trials tried during an experiment.

``pool-size``

The number of trials which are generated by the algorithm each time it is interrogated.

Results
=======


When an experiment reaches its termination criterion, basically ``max-trials``, it will print the
following statistics if Oríon is called with ``-v`` or ``-vv``.

.. code-block:: bash

    RESULTS
    =======
    {'best_evaluation': 0.05289999999999995,
     'best_trials_id': 'b7a741e70b75f074208942c1c2c7cd36',
     'duration': datetime.timedelta(0, 49, 751548),
     'finish_time': datetime.datetime(2018, 8, 30, 1, 8, 2, 562000),
     'start_time': datetime.datetime(2018, 8, 30, 1, 7, 12, 810452),
     'trials_completed': 5}

    BEST PARAMETERS
    ===============
    [{'name': '/lr', 'type': 'real', 'value': 0.012027705702344259}]


You can also fetch the results using python code. You do not need to understand MongoDB since 
you can fetch results using the ``Experiment`` object. The class `ExperimentBuilder` provides
simple methods to fetch experiments using their unique names. You do not need to explicitly 
open a connection to the database when using the `ExperimentBuilder` since it will automatically
infer its configuration from the global configuration file as when calling Oríon in commandline.
Otherwise you can pass other arguments to ``ExperimentBuilder().build_view_from()`` using the same
dictionary structure as in the configuration file.

.. code-block:: python

   # Database automatically inferred
   ExperimentBuilder().build_view_from(
       {"name": "orion-tutorial"})

   # Database manually set
   ExperimentBuilder().build_view_from(
       {"name": "orion-tutorial",
        "dataset": {
            "type": "mongodb",
            "name": "myother",
            "host": "localhost"}})

For a complete example, here's how you can fetch trials from a given experiment.

.. code-block:: python

   import datetime
   import pprint

   from orion.core.io.experiment_builder import ExperimentBuilder

   some_datetime = datetime.datetime.now() - datetime.timedelta(minutes=5)

   experiment = ExperimentBuilder().build_view_from({"name": "orion-tutorial"})

   pprint.pprint(experiment.stats)

   for trial in experiment.fetch_trials({}):
       print(trial.id)
       print(trial.status)
       print(trial.params)
       print(trial.results)
       print()
       pprint.pprint(trial.to_dict())

   # Fetches only the completed trials
   for trial in experiment.fetch_trials({'status': 'completed'}):
       print(trial.objective)
   
   # Fetches only the most recent trials using mongodb-like syntax
   for trial in experiment.fetch_trials({'end_time': {'$gte': some_datetime}}):
       print(trial.id)
       print(trial.end_time)

You can pass queries to ``fetch_trials()``, where queries can be simple dictionary of values to
match like ``{'status': 'completed'}``, in which case it would return all trials where
``trial.status == 'completed'``, or they can be more complex using `mongodb-like syntax`_.

.. _`mongodb-like syntax`: https://docs.mongodb.com/manual/reference/method/db.collection.find/


Research Iteration and Version Control
======================================

While doing your research your are very likely to iterate on ideas and change many things about your
experiments. You could obviously change the code, but also the hyper-parameters you want to optimize
or the algorithm you use to optimize them. In any other hyper-parameter optimization framework
you would start optimization from scratch each time. With Oríon comes an Experiment Version Control
(EVC) system which makes it possible to reuse results from your first experiment in a given project
to the current one. This means a new experiment could pre-train on all prior data resulting in a
much more efficient optimization algorithm. Another advantage of the EVC system is that it provides
you a systematic way to organize your research and the possibility to go back in time and compare
the evolution of performance throughout your research.

The EVC system is certainly the part of Oríon with the steepest learning curve. We tried to make it
such that you get help at any step and are provided with many hints so that you can learn while 
using it, without looking at the documentation. If you have any recommendation on how to
improve those hints and how to make the EVC system more intuitive, please do not hesitate to contact
us. You are very welcome to start a new thread on our Discourse_.

To continue with our examples from pytorch-mnist, suppose we decide at some point we would like to
also optimize the ``momentum``.

.. code-block:: bash

    $ orion hunt -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)' --momentum~'uniform(0, 1)'

This cannot be the same as the experiment ``orion-tutorial`` since the space of optimization is now
different. Such a call will trigger an experiment branching, meaning that a new experiment will
be created which points to the previous one, the one without momentum in this case.

.. code-block:: text

    Welcome to Orion's experiment branching interactive conflicts resolver
    -----------------------------------------------------------------------

    If you are unfamiliar with this process, you can type `help` to print the help message.
    You can also type `abort` or `(q)uit` at any moment to quit without saving.
    
    Remaining conflicts:
    
       Experiment name 'orion-tutorial' already exist for user 'bouthilx'
       New momentum

    (orion)

You should think of it like a ``git status``. It tells you want changed and what you did not commit
yet. If you hit tab twice, you will see all possible commands. You can enter `h` or `help` for more
information about each command. In this case we will first add ``momentum``. You can enter ``add``
and then hit tab twice. Oríon will detect any possible hyper-parameter that you could add and
autocomplete it. Since we only have ``momentum`` in this case, it will be fully autocompleted. If
you hit tab twice again, the option ``--default-value`` will be added to the line, with which you
can set a default-value for the momentum. If you only enter ``add momentum``, the new experiment
won't be able to fetch trials from the parent experiment, because it cannot know what was the
implicit value of ``momentum`` on those trials. If you know there was a default value 
for ``momentum``, you should tell so with ``--default-value``.


.. code-block:: text

    (orion) add momentum --default-value 0
    TIP: You can use the '~+' marker in place of the usual ~ with the command-line to solve this
    conflict automatically.
    Ex: -x~+uniform(0,1)

    Resolutions:

         momentum~+uniform(0, 1, default_value=0.0)

   
    Remaining conflicts:

         Experiment name 'orion-tutorial' already exist for user 'bouthilx'

    (orion)

As you can see, when resolving the conflicts with the prompt, Oríon will always tell you how
you could have resolved the conflict directly in commandline. If we follow the advice, we would
change our commandline like this.

.. code-block:: bash

    $ orion hunt -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)' --momentum~+'uniform(0, 1)'

Let's look back at the prompt above. Following the resolution of ``momentum`` conflict we see that it is now
marked as resolved in the `Resolutions` list, while the experiment name is still marked as a
conflict. Notice that the prior distribution is slightly different than the one specified in
commandline. This is because we added a default value inside the prompt. Notice also that the
resolution is marked as how you would resolve this conflict in commandline. There is hints
everywhere to help you learn without looking at the documentation.

Now for the experiment name conflict. Remember that experiment names must be unique, that means that
when an experiment branching occur we need to give a new name to the child experiment. You can do so
with the command ``name``. If you hit tab twice with ``name``, Oríon will auto-complete with all
experiment names in the current project. This makes it easy to autocomplete an experiment name and
simply append some version number like ``1.2`` at the end. Let's add ``-with-momentum`` in our case.

.. code-block:: text

    (orion) add orion-tutorial-with-momentum
    TIP: You can use the '-b' or '--branch' command-line argument to automate the naming process.

    Resolutions:

         --branch orion-tutorial-with-momentum
         momentum~+uniform(0, 1, default_value=0.0)

   
    Hooray, there is no more conflicts!
    You can enter 'commit' to leave this prompt and register the new branch


    (orion)

Again Oríon will tell you how you can resolve an experiment name conflict in command-line to avoid
the prompt, and the resolution will be marked accordingly.

.. code-block:: bash

    $ orion hunt -n orion-tutorial -b orion-tutorial-with-momentum ./main.py --lr~'loguniform(1e-5, 1.0)' --momentum~+'uniform(0, 1)'

You can execute again this branched experiment by reusing the same commandline but replacing the new
experiment name ``orion-tutorial-with-momentum``.

.. code-block:: bash

    $ orion hunt -n orion-tutorial-with-momentum ./main.py --lr~'loguniform(1e-5, 1.0)' --momentum~'uniform(0, 1)'

Or as always by only specifying the experiment name.

.. code-block:: bash

    $ orion hunt -n orion-tutorial-with-momentum

If you are unhappy with some resolutions, you can type ``reset`` and hit tab twice. Oríon will 
offer autocompletions of the possible resolutions to reset.

.. code-block:: text

    (orion) reset '
    '--branch orion-tutorial-with-momentum'
    'momentum~+uniform(0, 1, default_value=0.0)'
    (orion) reset '--branch orion-tutorial-with-momentum'

    Resolutions:

         momentum~+uniform(0, 1, default_value=0.0)

   
    Remaining conflicts:

         Experiment name 'orion-tutorial' already exist for user 'bouthilx'

    (orion)

Once you are done, you can enter ``commit`` and the branched experiment will be register and will
begin execution.

Note that all of this can be partially avoided using the option ``--auto-resolution`` in commandline
or ``auto`` in the interactive prompt. This will automatically resolve any conflict related to
hyper-parameters and algorithms. For now, Oríon cannot solve automatically experiment name
conflicts, code conflicts, command-line conflicts and configuration file conflicts.

.. code-block:: bash

    $ orion hunt --auto-resolution -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)' --momentum~'uniform(0, 1)'

Source of conflicts
-------------------

1. Code modification (not yet release, candidate for v0.1.1)
2. Commandline modification
3. Script configuration file modification
4. Optimization space modification (new hyper-parameters or change of prior distribution)
5. Algorithm configuration modification

Iterative Results
=================

You can retrieve results from different experiments in the same project using
the Experiment Version Control (EVC) system. The only difference 
with ``ExperimentBuilder`` is that ``EVCBuilder`` will connect the experiment
to the EVC system, accessible through the ``node`` attribute.

.. code-block:: python

   import pprint
   from orion.core.io.evc_builder import EVCBuilder

   experiment = EVCBuilder().build_view_from(
       {"name": "orion-tutorial-with-momentum"})

   print(experiment.name)
   pprint.pprint(experiment.stats)

   parent_experiment = experiment.node.parent.item
   print(parent_experiment.name)
   pprint.pprint(parent_experiment.stats)

   for child in experiment.node.children:
       child_experiment = child.item
       print(child_experiment.name)
       pprint.pprint(child_experiment.stats)


Deployment
==========

To run on Mila, you can simply call your script with Oríon as described in
the sections above. Here's an example for a bash script to run with sbatch.

.. code-block:: bash

	#!/usr/bin/env bash

	export HOME=`getent passwd $USER | cut -d':' -f6`

	source ~/.bashrc

	export PYTHONUNBUFFERED=1
	echo Running pwdon $HOSTNAME

	workon orion.tutorial
	orion hunt -n orion-tutorial ./main.py --lr~'loguniform(1e-5, 1.0)'
	
	# Or simply `orion hunt -n orion-tutorial` if already registered

You can launch as many times as you want the same experiment. Each time you execute an experiment,
it creates internally a worker which synchronizes with the others through the database. You don't
need to setup any master node for this.


Setup on Cedar
--------------

TODO

Plugins
=======

Oríon is built to be highly modular, so that algorithms and database backends can be
extended by installing plugins. So far there is only one plugin supported,
which is a Bayesian optimizer algorithm. We won't go into the details
of how to develop new algorithm or database backends during this tutorial+hackaton 
but if you are interested please do not hesitate to contact us and we will be glad to help.

Algorithms and database cannot be defined in commandline so you need to set
them in the configuration file. We recommend to configure the database in global
configuration files since the database are not likely to change from one experiment to
another. For algorithms however you should probably favor local configuration
files which are specified to Oríon with option ``--config``.

Bayesian Optimizer
------------------

Suppose we define the file ``bayes.yaml`` as this

.. code-block:: yaml

     name: orion-tutorial-with-bayes
     algorithms: BayesianOptimizer

Then with call ``orion hunt`` with the configuration file.

.. code-block:: bash

	$ orion hunt --config bayes.yaml ./script.sh --lr~'loguniform(1e-5, 1.0)'

And voilà, we have a Bayesian optimizer sampling learning-rate values to optimize the error rate.

--------------
Wait, what if?
--------------

There is a plethora of possible workflows and we obviously only presented one
such example in this tutorial. Because of that, many of you would be probably
tempted to ask *Wait, what if?*

Wait, what if I use a configuration file for my hyper-parameters?
=================================================================

You can use configuration files using the keywords ``'orion~dist(*args, **kwargs)'``. Oríon supports any
text based configuration file. Note that for now Oríon is a bit picky and
requires you to define the configuration file by passing it with
``--config=file_path`` where the ``=`` matters.

Here is an example with yaml

.. code-block:: yaml

    lr: 'orion~loguniform(1e-5, 1.0)'
    model:
      activation: 'orion~choices(['sigmoid', 'relu'])'
      hiddens: 'orion~randint(100, 1000)'

Here is another example with json

.. code-block:: json

    {
      "lr": "orion~loguniform(1e-5, 1.0)"
      "model": {
        "activation": "orion~choices(['sigmoid', 'relu'])"
        "hiddens": "orion~randint(100, 1000)"
      }
    }

And here is an example with python! Note that for other files than for json and yaml, the keywords
are defined as ``hp_name~dist(*args, **kwargs)``. Also, note that the code does not execute as is,
but once Oríon makes the substitution it will.

.. code-block:: python

    def my_config():
        lr = lr~loguniform(1e-5, 1.0)
        activations = model/activations~choices(['sigmoid', 'relu'])
        nhiddens = model/hiddens~randint(100, 1000)
        
        layers = []
        for layer in range(model/nlayers~randint(3, 10)):
            nhiddens /= 2
            layers.append(nhiddens)
        
        return lr, layers

Oríon could generate a script like this one for instance.

.. code-block:: python

    def my_config():
        lr = 0.001
        activations = 'relu'
        nhiddens = 100
        
        layers = []
        for layer in range(4):
            nhiddens /= 2
            layers.append(nhiddens)

        return lr, layers


Wait, what if I execute from bash script, not python?
=====================================================

If you pass your arguments to the script, something like

.. code-block:: bash

	python somescript.py $@

Then Oríon can work flawlessly, because it only interacts with the arguments of the script.

.. code-block:: bash

	$ orion hunt -n orion-tutorial ./script.sh --lr~'loguniform(1e-5, 1.0)' --momentum~+'uniform(0, 1)'

If you define the arguments within the script, then you would need to call orion from within the script like
we did for sbatch in section Deployment.

Wait, what if I'm not using python?
===================================

Wait, really?

Suppose you do have a configuration file for the arguments or use command line arguments. Then
the only thing you need to do is to get the value of the environment variable ``ORION_RESULTS_PATH``
and write to it a json file following this architecture.

.. code-block:: json

	[
	  {
	    "name": "my_objective_function",
	    "type": "objective",
	    "value": 0.0
	  }
	]

This is basically what is doing the helper function ``orion.client.report_results``. 


Wait, what if I want to add points to the search?
=================================================

If you are asking because you intend to do grid search, then you should take a pause and think about
why you would want to do grid search. Grid search is highly inefficient, even random search is
better. Best solution if you want broad analysis is to rather use an optimization algorithm to focus
on regions of hyper-parameter space where it affects performance and afterwards use an estimator on
top of the results to interpolate performance. This is current practice in literature such as
hyper-parameter importance analysis.

Otherwise, adding points based on our prior knowledge of good hyper-parameter values is a good
way to warm-start an optimization algorithm. To do so, you can use the command ``insert`` 
following the same template as for ``hunt`` but using specific values rather than prior 
distribution. Note that for now ``insert`` only supports argument specifications using ``=`` sign.
It will fail otherwise.

.. code-block:: bash

	$ orion insert -n orion-tutorial ./script.sh --lr=0.01 --momentum=0.9

Suppose you have many hyper-parameters and you would like to specify just a few when adding points
manually. You can do so if the hyper-parameters you ignore have default values defined in the
experiment.

.. code-block:: bash

	$ orion hunt -n orion-tutorial ./script.sh --lr~'loguniform(1e-5, 1.0)' --momentum~'uniform(0, 1, default_value=0)'

.. code-block:: bash

	$ orion insert -n orion-tutorial ./script.sh --lr=0.01

When points are added manually, they are saved in the database and may be picked up for execution in
a latter call to ``orion hunt -n orion-tutorial``. The command ``insert`` does not execute trials.


Wait, what if I use python2.7?
==============================

C'mon, python2.7 is 8 years old. Install python3.6+ 😉

Still hesitating? Do you know numpy is planning to drop_ full support for python2 on January 1, 2019?

.. _drop: https://docs.scipy.org/doc/numpy-1.14.0/neps/dropping-python2.7-proposal.html


Wait, what if I use windows?
============================

Well, maybe you should consider clicking here_. 😁

.. _here: https://www.ubuntu.com/#download

.. _Discourse: https://discourse-epistimio.mila.quebec
.. _github: https://github.com/epistimio/orion
.. _slack: https://mila-umontreal.slack.com/messages/CABLBMPHD/
.. _kleio: https://github.com/epistimio/kleio
