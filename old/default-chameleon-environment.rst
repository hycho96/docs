Default Chameleon Environment
=============================

The purpose of the Default Chameleon Environment is to provide a useful
standard OpenStack instance that will get folks started using the cloud
with the minimum of effort.

The default image is called CC-CentOS7.

Quickstart
----------

First start up an instance on the Chameleon Alamo cloud. See
instructions here: `Alamo User Guide <alamo-user-guide.md>`__.

Login to that instance and create an openrc file using your username &
tenant. It should look something like this:

::

    export OS_AUTH_URL=http://ironic.chameleon.tacc.utexas.edu:5000/v2.0
    export OS_TENANT_NAME="mpackard_studies"
    export OS_USERNAME="mpackard"
    echo "Please enter your OpenStack Password: "
    read -sr OS_PASSWORD_INPUT
    export OS_PASSWORD=$OS_PASSWORD_INPUT
    export OS_REGION_NAME="regionOne"
    if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi

Source that file and enter your password when prompted. You should now
be able to run *nova*, *glance*, and *neutron* commands:

::

    # nova list 
    | d79c9e97-607e-448d-b569-f71cddf5390e | CC-CentOS7-46 | ACTIVE | -          | Running     | sharednet1=10.12.0.52, 129.114.34.12 |

    # glance image-list
    | 2d7f7712-346a-4186-bbce-ad653fe4e5d1 | CC-CentOS7        | qcow2       | bare             | 542113792 | active |
    | b39feeff-63f5-44ad-b1ff-7cf70d1a4309 | CC-CentOS7.initrd | ari         | ari              | 30696161  | active |
    | a0aa7bde-c5a3-4180-b64d-0032e2da168e | CC-CentOS7.kernel | aki         | aki              | 4906464   | active |

    # neutron net-list
    | d4e6c5e0-0477-4efd-bd2e-a25ccc960de7 | sharednet1 | dea413d0-9657-4400-aa2c-94af908442d8 10.12.0.0/24 |
    | f31afa90-df73-4b10-af55-528a3c60431c | ext-net    | 87fbe48d-6771-40e6-992e-7a250436c9fb              |

And boot an instance on bare metal:

::

    # nova boot --flavor my-baremetal-flavor --image CC-CentOS7 --key-name default --nic net-id=d4e6c5e0-0477-4efd-bd2e-a25ccc960de7 my-centos-image

Example Benchmark Operation
---------------------------

Create an instance per the Quickstart section (above).

SSH into the instance.

Install sysbench:

::

    # sudo yum install -y sysbench iperf3

Determine how many CPUs the node has:

::

    # cat /proc/cpuinfo  | grep ^processor | wc -l
    8

CPU Test
~~~~~~~~

Do a cpu test, using up to the number of CPUs available. Adjust the
*--cpu-max-prime* number if the test runs too short or too long.

::

    # sysbench --test=cpu --cpu-max-prime=20000 --num-threads=8 run
    ...

IO Test
~~~~~~~

Make sure you have enough disk space free. Then run an IO test.

::

    # df -h /
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda1       438G  103G  318G  25% /
    # sysbench --test=fileio --file-total-size=100G --num-threads=8 prepare
    ...
    # sysbench --test=fileio --file-total-size=100G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
    ...

After doing your tests, remove the test file:

::

    # sysbench --test=fileio --file-total-size=150G cleanup
    ...
