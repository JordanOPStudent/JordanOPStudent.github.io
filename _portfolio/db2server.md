---
layout: post
title: DB2 Server
thumbnail-path: "img/db2.png"
short-description: Performing a Lift and Shift of the existing DB2 server to a virtualized environment.

---

{:.center}
![]({{ site.baseurl }}/img/db2.png)

## Explanation

The BIT department of the Otago Polytechnic are looking to move all of their physical servers onto a virtualized environment with Microsoft Azure. This is part of an ongoing movement to shift everything to a more flexible and easier to maintain platform as a service model. One of those servers is our MariaDB server that's used for the Databases 2 (DB2) classes that teaches MySQL and will be actively used by the students.

## Problem
DB2 is expensive to run. The cost of power, backup disks, maintenance. A lot of it adds up for hosting our own server. On top of that, we have a smaller security server to test various things on that replicates all of the databases on the main DB2 server that will need to be migrated. Our goal is to seamlessly integrate this infrastructure over to the Azure platform. At the end of this project, this server should be able to fit into the classes without any issues. This is meant to be a drop in replacement for the current server so we may decommission  the physical servers.

## Solution

Our solution is to perform a "lift and shift" method, which is to simply remake the environment of the current server on a new virtual machine. We have considered moving our infrastructure to like Azure's own SQL platforms which allows the management of that through Azure but there was two problems.
- One, we'd have no front end interface for phpMyAdmin, one of the tools needed for administrating the databases used by the staff.
- Two, the students will need another method to use the mysql terminal as the class relies on SSHing into MariaDB to connect to the terminal.

Considering our other options, we decided to simply replicate the environment of the MariaDB server into a new Virtual Machine, clone it for the security server and configure everything to replicate the current server as much as possible.

## Results

While we intended to have everything sorted out during one term, it ended up being far more complicated than expected. Due to a mixture of my inexperience with Microsoft Azure and some organizational issues, we did make up for it with a lot of polishing and automation. My colleague handled most of the planning and proposal work, I worked on most of the technical side and getting the server running.

We ended up encountering a few issues. One was the backup solution. Due to how we were using one disk for the OS and root while we used another for the database and other miscellanious things, it turns out that we did not have permissions to use Azure's services to back up the database disk nor did we have the subscription to use that. So, I had to get crafty and use what services we could use. Our solution to the backup problem was to use an Azure container (which to me was similar to an Amazon Web Service Bucket), perform a full database backup using mariabackup, mount the container then upload it to the container.
```bash
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
			blobfuse /media/blob/ --tmp-path=/mnt/blobtmp \
			--config-file=/etc/azureblob/azblob.cfg \
			-o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120

			# Compresses and archives the backup to save on storage
			tar caf "${BASE_PATH}/${TIMESTAMP}.tar.xz" "${FULL_PATH}"
			# Moves it over to the container so it may be uploaded.
			mv "${BASE_PATH}/${TIMESTAMP}.tar.xz" /media/blob/

			# Dismounts the container so it may be used for later.
			fusermount -u /media/blob/
	}

	createbackup
	mountblob
```
Another issue we had was integrating everything into Azure. This was where my inexperience with the platform came into play. At first, my solution was to use the standard Debian package `unattended-upgrades` but it turned out Azure have their own solution. Removing that, I then proceeded to set up how Azure do it. Turns out, I didn't have permissions to create the automation system to roll that. So, I had to ask my supervisor to set that up for me. Once that had been sorted out, it was easy going from there. There was a lot I had to learn with how Azure did things as most of my experience comes from using the simple tools that aren't used for platforms such as Azure.

Something I got far too invested in was the documentation for the installation process. As I wanted it to be verbose enough that if you had no idea how to set up anything in the server, by the time you finish it, you'd have finished it all up. What I didn't expect was how verbose I became with it. I don't want to say the documentation was 100% perfect as there was just too much to write about but I hope it was enough. I hope that after we are finished with this project that whoever might take over will understand how everything works.

![]({{ site.baseurl }}/img/overkill.png)

There was also a lot of issues just with my unfamiliarity with the current DB2 server that caused a lot of problems. This was due to a lot of communication problems that were entirely my fault. I was a bit hesitant in asking as I was not familiar with the team nor did I have too much confidence in asking about simple questions. That has been something that's been improved upon during the latter parts of this project.


## Conclusion

This project was quite a learning experience for me. There's several things I need work on. How to divy up my time as I took on a lot more work than I could handle, work I should split between me and my colleague. I should've been a bit more open with the rest of the team and asking for help as I have a tendency to hole up on my own and just crack at it away myself. I should work on being more familiar with the Azure workflow and their myriad of systems as having someone else help me out would have greatly assisted with that (although I should have asked). And I should be more open to simply ask about things and simply communicate. Splitting up my work to my team is something I need to greatly look into as although my colleague has done quite a lot of work on this project as well, once it got to a certain point I simply took over and became consumed in it. I'll be taking these issues into account for my next project going forward and working on improving my communication and organizational skills.