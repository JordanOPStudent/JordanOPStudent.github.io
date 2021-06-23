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

Bacon ipsum dolor amet filet mignon meatball spare ribs fatback bacon shankle. Kielbasa andouille fatback salami, boudin bresaola pig alcatra turkey spare ribs jerky. Corned beef bresaola leberkas salami alcatra beef landjaeger venison shank bacon meatloaf beef ribs picanha. Leberkas sausage brisket porchetta shankle prosciutto chicken picanha kielbasa pig kevin t-bone turducken filet mignon jowl.

> Bacon ipsum dolor amet filet mignon meatball spare ribs fatback bacon shankle. Kielbasa andouille fatback salami, boudin bresaola pig alcatra turkey spare ribs jerky. Corned beef bresaola leberkas salami alcatra beef landjaeger venison shank bacon meatloaf beef ribs picanha. Leberkas sausage brisket porchetta shankle prosciutto chicken picanha kielbasa pig kevin t-bone turducken filet mignon jowl.

Bacon ipsum dolor amet filet mignon meatball spare ribs fatback bacon shankle. Kielbasa andouille fatback salami, boudin bresaola pig alcatra turkey spare ribs jerky. Corned beef bresaola leberkas salami alcatra beef landjaeger venison shank bacon meatloaf beef ribs picanha. Leberkas sausage brisket porchetta shankle prosciutto chicken picanha kielbasa pig kevin t-bone turducken filet mignon jowl.

## Conclusion

Bacon ipsum dolor amet filet mignon meatball spare ribs fatback bacon shankle. Kielbasa andouille fatback salami, boudin bresaola pig alcatra turkey spare ribs jerky. Corned beef bresaola leberkas salami alcatra beef landjaeger venison shank bacon meatloaf beef ribs picanha. Leberkas sausage brisket porchetta shankle prosciutto chicken picanha kielbasa pig kevin t-bone turducken filet mignon jowl.