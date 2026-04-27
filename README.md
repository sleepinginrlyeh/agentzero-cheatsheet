# agentzero-cheatsheet
User-contributed "how to" guide for AgentZero

This is a user-contributed how to guide for using AgentZero


# Quick Reminders
Agent Zero runs in a container, so any files changed anywhere in the container except under /a0/usr will be blown away.  /a0/usr should generally be bound to a directory that you have rw access to.
Agent Zero runs as root so when it writes files to /a0/usr, you will not have write access by default.  See section [Write Permissions for Shared Files](#write-permissions-for-shared-files)





## Write Permissions for Shared Files
Since Agent Zero runs as root in the container, it will write files / directories with rw-r--r--  This makes it hard to use when, fore example, one wants to access the same git files as A0.

There is no "good" solution, but the easiest is to create a group, say "agentzero", join the group, and use the setgid bit to always change the ownership of files to the group.

These instructions assume you have ~/agent-zero bound to /a0/usr in your container.  Adjust as necessary.

### Create an AgentZero group
sudo group add agentzero
sudo usermod -aG agentzero <yourusername> 

### Change ownership of all current files to the group
sudo chgrp -R agentzero ~/agent-zero

### Use the setgid bit so all files written are set to group "agentzero" - do this recursively
sudo find ~/agent-zero -type d -exec chmod g+s {} +

### Change existing permissions so group has read-write access
sudo chmod -R g+rw ~/agent-zero

There is one missing part.  The default UMASK used by the agentzero container is 022 (rw-r--r--).  So when Agent Zero creates a new file the group will not have write access. You'll need to periodically refresh the permissions.  However, already created files will keep the group rw permissions:
sudo chmod -R g+rw ~/agent-zero


