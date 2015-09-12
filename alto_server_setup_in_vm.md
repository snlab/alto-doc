# ALTO Server Setup in VM
---

## Part I: Deployment of ALTO Server on VM

### note:
We have bulit a VM with the ALTO server and taken the snapshot. If you use the snapshot to build a VM, you can skip this part and turn to [Part II: Run ALTO Server](#partII).

### Create Virtual Machine
* Create a VM and install the operating system (Here we use VMware to create VM and Ubuntu Server 14 for operating system).

<span id = "partI"></span>
### Compile and Install ALTO

* [Development environment setup for ODL](https://wiki.opendaylight.org/view/GettingStarted:Development_Environment_Setup).

#### note:
The link includes installation of java jdk, maven and git. Also don't forget to edit your ~/.m2/settings.xml file, otherwise you will not be able to download any ODL Java dependencies.

* Clone ALTO project from ODL [Gerrit](https://git.opendaylight.org/gerrit/#/admin/projects/alto) or [Github](https://github.com/opendaylight/alto).

	Gerrit:

		$ git clone https://git.opendaylight.org/gerrit/alto
		
	Github:

		$ git clone https://github.com/opendaylight/alto.git
	

* Enter your ALTO project root directory and run `mvn compile` or `mvn install` to compile or install ALTO project.

		$ cd $ALTO_ROOT/
		$ mvn install

### Run ALTO in Karaf 
* Enter your ALTO project root directory.

* Run ALTO in karaf.
	
		$ ./alto-karaf/target/assembly/bin/karaf

	![alto_karaf](http://ww1.sinaimg.cn/mw1024/006asSVejw1eu0bt7j302j314009s75u.jpg)

* You can now use the ALTO server.

### ALTO Server as Service
Using the script below to run ALTO server as service.

#### Script:

	#!/bin/sh
	### BEGIN INIT INFO
	# Provides:          <NAME>
	# Required-Start:    $local_fs $network $named $time $syslog
	# Required-Stop:     $local_fs $network $named $time $syslog
	# Default-Start:     2 3 4 5
	# Default-Stop:      0 1 6
	# Description:       <DESCRIPTION>
	### END INIT INFO

	SCRIPT=/home/alto/code/alto-karaf-0.2.0-SNAPSHOT/bin/karaf
	RUNAS=alto

	PIDFILE=/home/alto/alto.pid
	LOGFILE=/home/alto/alto.log

	start() {
		if [ -f /var/run/$PIDNAME ] && kill -0 $(cat /var/run/$PIDNAME); then
    		echo 'Service already running' >&2
    		return 1
		fi
		echo 'Starting service…' >&2
		local CMD="$SCRIPT &> \"$LOGFILE\" & echo \$!"
		su -c "$CMD" $RUNAS > "$PIDFILE"
		echo 'Service started' >&2
	}

	stop() {
		if [ ! -f "$PIDFILE" ] || ! kill -0 $(cat "$PIDFILE"); then
    		echo 'Service not running' >&2
    		return 1
    	fi
    	echo 'Stopping service…' >&2
    	kill -15 $(cat "$PIDFILE") && rm -f "$PIDFILE"
    	echo 'Service stopped' >&2
	}

	uninstall() {
		echo -n "Are you really sure you want to uninstall this service? That cannot be undone. [yes|No] "
		local SURE
		read SURE
		if [ "$SURE" = "yes" ]; then
			stop

	▽
    	rm -f "$PIDFILE"
    	echo "Notice: log file is not be removed: '$LOGFILE'" >&2
    	update-rc.d -f <NAME> remove
    	rm -fv "$0"
		fi
	}

	case "$1" in
		start)
    	  start
    	  ;;
		stop)
    	  stop
    	  ;;
		uninstall)
    	  uninstall
    	  ;;
		retart)
    	  stop
    	  start
    	  ;;
		*)
    	  echo "Usage: $0 {start|stop|restart|uninstall}"
	esac

* Move the script to the directory /etc/init.d/.

* Update the startup service.

		$ cd /etc/init.d/
		$ update-rc.d
		
* Verify that your ALTO server has been started.

		$ ps aux | grep karaf
		
	You can see kara is running.
	
	![grep karaf](http://ww3.sinaimg.cn/mw1024/006asSVejw1etzvwglgf5j313z09dqe9.jpg)

###Taking a Snapshot for the VM
* Use VMware snapshot function to take a snapshot.

### Optional:ALTO Karaf Configuration
Change the username and password for karaf for security.

* Edit ~/code/alto-karaf-0.2.0-SNAPSHOT/etc/users.properties. Change the default username and password to your own username and password.

	e.g.

		yale-alto-server = tongji-yale,_g_:admingroup
		_g_\:admingroup = group,admin,manager,viewer,webconsole


<span id = "partII"></span>
## Part II: Run ALTO Server

### Run ALTO server on Virtual Machine
* Enter the virtual machine you  built.

* Enter your ALTO project root directory.

* Run karaf.
	
		$ ./alto-karaf/target/assembly/bin/karaf

### Remote Access to ALTO Server by Karaf
You can access the ALTO server from your local machine.

* You need to install ALTO on your local machine, see [Part I: Compile and Install ALTO](#partI).

* Run your local karaf.
		
		$ cd $ALTO_ROOT/alto-karaf/target/assembly/bin/
		$ ./karaf
		
* Access to ALTO server.

		$ ssh:ssh -l <user_name> -P <password> -p <port_number> <server_ip>
		(e.g.ssh:ssh -l yale-alto-server -P tongji-yale -p 8101 192.168.1.120)


