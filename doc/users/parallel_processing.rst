.. _parallel_processing:

====================================
 Distributed processing with nipype
====================================

The workflow engine is designed to support plugin interfaces for
distributed processing. Current plugins are available for IPython_ (0.10.1
or higher) distributed processing platforms and for direct processing on
SGE/OGE. We anticipate future plugins for multiprocessing, the Soma_
workflow, PBS_, LSF_ and Condor_.

Parallel distributed processing relies on the availability of a shared
filesystem across computational nodes.

Using the pipeline engine with IPython
--------------------------------------

The pipeline engine provides a mechanism to distribute processes across
multiple cores and machines in a cluster employing a consistent login
system and a shared file system. Currently, the login process needs to
be ssh-able via public key authentication. This document now reflects use 
with IPython_ 0.10.1 or higher.

Please read the IPython_ documentation to determine how to setup your
cluster for distributed processing. This typically involves calling
ipcluster. For example the following command will start an eight client
cluster locally and log all client messages to the file in
/tmp/pipeline::

        ipcluster local -n 8 --logdir /tmp/pipeline
        
If you use a more complicated environment distributed over ssh try using the following configuration::

        ipcluster ssh -e --clusterfile=clusterfile.py

clusterfile.py example::

    send_furl = False
    # define cluster configurations
    half_cores = { 'xxx.mit.edu' : 4,
                          'yyy.mit.edu' : 4,
                          'zzz.mit.edu' : 4}
    all_cores = { 'xxx.mit.edu' : 4,
                       'yyy.mit.edu' : 8,
                       'zzz.mit.edu' : 8}
    xxx_only = { 'xxx.mit.edu' : 4}
    # choose cluster configurations
    engines = all_cores # this is the primary information that ipcluster needs

Once the clients have been started, any pipeline executed with::

 workflow.run(plugin=IPython)

will automatically start getting distributed to the
clients. Alternatively, a config file may be used to define the
plugin. See :ref:`config_file_` for details.

To prevent prevent parallel execution type::

    workflow.run(plugin=Linear)

Using the pipeline engine with SGE/OGE
--------------------------------------

In order to use nipype with SGE_/OGE_ (not tested), you simply need to
call::

       workflow.run(plugin=SGE)
 
you can also pass additional arguments to the SGE plugin through the
keyword argument (plugin_args). Currentyl the SGE engine, supports
sending a dictionary containing any of the following keys::

 queue - define which queue to run on
 template - custom template file. by 
 args - any other command line args to be passed to qsub.

For example, the following snippet executes the workflow on myqueue with
a custom template:
 
       workflow.run(plugin=SGE,
          plugin_args=dict(template='mytemplate.sh', queue='myqueue', args='-V')

.. include:: ../links_names.txt

.. _SGE: www.oracle.com/us/products/tools/oracle-grid-engine-075549.html
.. _OGE: www.oracle.com/us/products/tools/oracle-grid-engine-075549.html
.. _Soma: http://brainvisa.info/soma/soma-workflow/
.. _PBS: http://www.clusterresources.com/products/torque-resource-manager.php
.. _LSF: http://www.platform.com/Products/platform-lsf
.. _Condor: http://www.cs.wisc.edu/condor/
