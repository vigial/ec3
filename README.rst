.. image:: doc/EC3-logo-3d.png
   :height: 50px
   :width: 41 px
   :scale: 50 %
   :alt: alternate text
   :align: right
   
.. Elastic Cloud Computing Cluster (EC3)
=====================================

Elastic Cloud Computing Cluster (EC3) is a tool to create elastic virtual clusters on top
of Infrastructure as a Service (IaaS) providers, either public (such as `Amazon Web Services`_,
`Google Cloud`_ or `Microsoft Azure`_)
or on-premises (such as `OpenNebula`_ and `OpenStack`_). We offer recipes to deploy `TORQUE`_
(optionally with `MAUI`_) and `SLURM`_ clusters that can be self-managed with `CLUES`_:
it starts with a single-node cluster and working nodes will be dynamically deployed and provisioned
to fit increasing load, in terms of the number of jobs at the LRMS (Local Resource Management System). Working nodes will be undeployed when they are idle.
This introduces a cost-efficient approach for Cluster-based computing.
   
Installation
------------

The program `ec3` requires Python 2.6+, `PLY`_, `PyYAML`_ and an `IM`_ server, which is used to
launch virtual machines. By default `ec3` uses our public `IM`_ server in
`servproject.i3m.upv.es`. *Optionally* you can deploy a local `IM`_ server executing the
next commands::

    sudo pip install im
    sudo service im start

`PyYAML`_ and `PLY`_ are usually available in distribution repositories (``python-yaml``, ``python-ply`` in Debian; ``PyYAML``, ``python-ply`` in Red Hat; and ``PyYAML``, ``PLY`` in pip).

`ec3` can be download from `this <https://github.com/grycap/ec3>`_ git repository::

   git clone https://github.com/grycap/ec3

In the created directory there is the python executable file ``ec3``, which provides the
command-line interface described next.

Basic example with Amazon EC2
-----------------------------

First create a file ``auth.txt`` with a single line like this::

   id = provider ; type = EC2 ; username = <<Access Key ID>> ; password = <<Secret Access Key>>

Replace ``<<Access Key ID>>`` and ``<<Secret Access Key>>`` with the corresponding values
for the AWS account where the cluster will be deployed. It is safer to use the credentials
of an IAM user created within your AWS account.

This file is the authorization file (see `Authorization file`_), and can have more than one set of credentials.

The next command deploys a `TORQUE`_ cluster based on an `Ubuntu`_ image::

   $ ec3 launch mycluster torque ubuntu-ec2 -a auth.txt -y
   WARNING: you are not using a secure connection and this can compromise the secrecy of the passwords and private keys available in the authorization file.
   Creating infrastructure
   Infrastructure successfully created with ID: 60
      ▄▟▙▄¨        Front-end state: running, IP: 132.43.105.28

If you deployed a local `IM`_ server, use the next command instead::

   $ ec3 launch mycluster torque ubuntu-ec2 -a auth.txt -u http://localhost:8899

This can take several minutes.

Bear in mind that you have to specify a resource manager (like ``torque`` in our example) in addition to the images that you want to deploy (e.g. ``ubuntu-ec2``). For more information about this check the `templates documentation`_.

You can show basic information about the deployed clusters by executing::

    $ ec3 list
        name       state          IP        nodes
     ---------------------------------------------
      mycluster  configured  132.43.105.28    0

Once the cluster has been deployed, open a ssh session to the front-end (you may need to install the ``sshpass`` library)::

   $ ec3 ssh mycluster
   Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-24-generic x86_64)
   Documentation:  https://help.ubuntu.com/
   ubuntu@torqueserver:~$

You may use the cluster as usual, depending on the LRMS.
For Torque, you can decide to submit a couple of jobs using qsub, to test elasticity in the cluster:

   $ for i in 1 2; do echo "/bin/sleep 50" | qsub; done

Notice that CLUES will intercept the jobs submited to the LRMS to deploy additional working nodes if needed.
This might result in a customizable (180 seconds by default) blocking delay when submitting jobs when no additional working nodes are available.
This guarantees that jobs will enter execution as soon as the working nodes are deployed and integrated in the cluster.

Working nodes will be provisioned and relinquished automatically to increase and decrease the cluster size according to the elasticity policies provided by CLUES.

Enjoy your virtual elastic cluster!

Additional information
----------------------

* `EC3 Command-line Interface`_.
* `Templates`_.
* Information about available templates: ``ec3 templates [--search <topic>] [--full-description]``.

.. _`CLUES`: http://www.grycap.upv.es/clues/
.. _`RADL`: http://www.grycap.upv.es/im/doc/radl.html
.. _`TORQUE`: http://www.adaptivecomputing.com/products/open-source/torque
.. _`MAUI`: http://www.adaptivecomputing.com/products/open-source/maui/
.. _`SLURM`: http://slurm.schedmd.com/
.. _`Scientific Linux`: https://www.scientificlinux.org/
.. _`Ubuntu`: http://www.ubuntu.com/
.. _`OpenNebula`: http://www.opennebula.org/
.. _`OpenStack`: http://www.openstack.org/
.. _`Amazon Web Services`: https://aws.amazon.com/
.. _`Google Cloud`: http://cloud.google.com/
.. _`Microsoft Azure`: http://azure.microsoft.com/
.. _`IM`: https://github.com/grycap/im
.. _`PyYAML`: http://pyyaml.org/wiki/PyYAML
.. _`PLY`: http://www.dabeaz.com/ply/
.. _`EC3 Command-line Interface`: http://ec3.readthedocs.org/en/devel/ec3.html
.. _`Command templates`: http://ec3.readthedocs.org/en/devel/ec3.html#command-templates
.. _`Authorization file`: http://ec3.readthedocs.org/en/devel/ec3.html#authorization-file
.. _`Templates`: http://ec3.readthedocs.org/en/devel/templates.html
.. _`templates documentation`: http://ec3.readthedocs.org/en/devel/templates.html#ec3-types-of-templates
