---
layout: post
title: DB2 Server
thumbnail-path: "img/db2.png"
short-description: Performing a Lift and Shift of the existing DB2 server to a virtualized envrionment.

---

{:.center}
![]({{ site.baseurl }}/img/db2.png)

## Explanation

The BIT department of the Otago Polytechnic are looking to move all of their physical servers onto a virtualized environment with Microsoft Azure. This is part of an ongoing movement to shift everything to a more flexible and easier to maintain platform as a service model. One of those servers is our MariaDB server that's used for the Databases 2 (DB2) classes that teaches MySQL and will be actively used by the students.

## Problem
DB2 is expensive to run. The cost of power, backup disks, maintenance. A lot of it adds up for hosting our own server. On top of that, we have a smaller security server to test various things on that replicates all of the databases on the main DB2 server that will need to be migrated. Our goal is to seamlessly integrate this infrastructure over to the Azure platform. At the end of this project, this server should be able to fit into the classes without any issues. This is meant to be a drop in replacement for the current server so we may decomission the physical servers.

## Solution

Our solution is to perform a "lift and shift" method, which is to simply remake the environment of the current server on a new virtual machine. We have considered moving our infrastructure to like Azure's own SQL platforms which allows the management of that through Azure but there was two problems.
- One, we'd have no front end interface for phpMyAdmin, one of the tools needed for administrating the databases used by the staff.
- Two, the students will need another method to use the mysql terminal as the class relies on SSHing into MariaDB to connect to the terminal.

Considering our other options, we decided to simply replicate the environment of the MariaDB server into a new Virtual Machine, clone it for the security server and configure everything to replicate the current server as much as possible.

## Results

While we intended to have everything sorted out during one term, it ended up being far more complicated than expected. Due to a mixture of my inexperience with Microsoft Azure and some organizational issues, we did make up for it with a lot of polishing and automation. My colleague handled most of the planning and proposal work, I worked on most of the technical side and getting the server running.

We ended up encountering a few issues. One was the backup solution. Due to how we were using one disk for the OS and root while we used another for the database and other miscellanious things, it turns out that we did not have permissions to use Azure's services to back up the database disk nor did we have the subscription to use that. So, I had to get crafty and use what services we could use. Our solution to the backup problem was to use an Azure container (which to me was similar to an Amazon Web Service Bucket), perform a full database backup using mariabackup, mount the container then upload it to the container.

	#!/bin/bash

	TIMESTAMP=$(date +%Y%m%d)
	BASE_PATH="/var/mariadb/backup"
	FULL_PATH="${BASE_PATH}/full"

	function createbackup {
			# Removes all backups so we can start with a clean fresh slate
			# This assumes that the previous database has been backed up.
			rm -r "${BASE_PATH}"
			mkdir -p "${FULL_PATH}"

			# Create the initial full backup
			mariabackup --backup \
					--target-dir="${FULL_PATH}"

			# Create our first incremental backup so every other one can follow it.
			mkdir "${BASE_PATH}/${TIMESTAMP}"
			mariabackup --backup \
					--target-dir="${BASE_PATH}/${TIMESTAMP}" \
					--incremental-basedir="${FULL_PATH}"
	}


	function mountblob {
			# Mounts the container to the system. Needs error checking.
			blobfuse /media/blob/ --tmp-path=/mnt/blobtmp --config-file=/etc/azureblob/azblob.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120

			# Compresses and archives the backup to save on storage
			tar caf "${BASE_PATH}/${TIMESTAMP}.tar.xz" "${FULL_PATH}"
			# Moves it over to the container so it may be uploaded.
			mv "${BASE_PATH}/${TIMESTAMP}.tar.xz" /media/blob/

			# Dismounts the container so it may be used for later.
			fusermount -u /media/blob/
	}

	createbackup
	mountblob

Another issue we had was integrating everything into Azure. This was where my inexperience with the platform came into play. At first, my solution was to use the standard Debian package `unattended-upgrades` but it turned out Azure have their own solution. Removing that, I then proceeded to set up how Azure do it. Turns out, I didn't have permissions to create the automation system to roll that. So, I had to ask my supervisor to set that up for me. Once that had been sorted out, it was easy going from there. There was a lot I had to learn with how Azure did things as most of my experience comes from using the simple tools that aren't used for platforms such as Azure.




## Conclusion

Bacon ipsum dolor amet filet mignon meatball spare ribs fatback bacon shankle. Kielbasa andouille fatback salami, boudin bresaola pig alcatra turkey spare ribs jerky. Corned beef bresaola leberkas salami alcatra beef landjaeger venison shank bacon meatloaf beef ribs picanha. Leberkas sausage brisket porchetta shankle prosciutto chicken picanha kielbasa pig kevin t-bone turducken filet mignon jowl.