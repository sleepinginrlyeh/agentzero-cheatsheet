# agentzero-cheatsheet
This is a user-contributed how to guide for using AgentZero
<br/><br/>

# Help Resources
* Agent Zero [Discord Server](https://discord.gg/dpSzUPnX)
* Agent Zero Homepage [https://www.agent-zero.ai/](https://www.agent-zero.ai/)
* Agent Zero DeepWiki (AI RAG of source tree) [https://deepwiki.com/agent0ai/agent-zero](https://deepwiki.com/agent0ai/agent-zero)
* Agent Zero Documentation [https://www.agent-zero.ai/p/docs/](https://www.agent-zero.ai/p/docs/)

# Contents:
1. [Quick Reminders](#quick_reminders)
2. [Sharing Files With Agent Zero](#sharing-files-with-agent-zero)
3. [Using Google Vertex](#using-google-vertex)
<br/><br/>

# Quick Reminders
- Agent Zero runs in a container, so any files changed anywhere in the container except under /a0/usr will be blown away on an update or image pull.
- /a0/usr should generally be bound to a directory that you have rw access to.
- Agent Zero runs as root so when it writes files to /a0/usr, you will not have write access by default.
- See section [Write Permissions for Shared Files](#write-permissions-for-shared-files)
<br/><br/>

# Sharing Files With Agent Zero
## Write Permissions for Shared Files
Since Agent Zero runs as root in the container, it will write files and directories using the default UMASK of 022 (rw-r--r--))  This makes it hard to use when, fore example, one wants to access the same git files as A0.

There is no "good" solution, but the easiest is to create a group, say "agentzero", join the group, and use the setgid bit to always change the ownership of files to the group.

These instructions assume you have ~/agent-zero bound to /a0/usr in your container.  Adjust as necessary.

### Create an AgentZero group
```bash
sudo group add agentzero
sudo usermod -aG agentzero <yourusername>
```

### Change ownership of all current files to the group
```bash
sudo chgrp -R agentzero ~/agent-zero
```

### Use the setgid bit so all files written are set to group "agentzero" - do this recursively
```bash
sudo find ~/agent-zero -type d -exec chmod g+s {} +
```

### Change existing permissions so group has read-write access
```bash
sudo chmod -R g+rw ~/agent-zero
```

There is one missing part.  The default UMASK used by the agentzero container is 022 (rw-r--r--).  So when Agent Zero creates a new file the group will not have write access. You'll need to periodically refresh the permissions.  However, already created files will keep the group rw permissions:
```bash
sudo chmod -R g+rw ~/agent-zero
```

There is a pending ticket requesting UMASK support: [https://github.com/agent0ai/agent-zero/issues/1584](https://github.com/agent0ai/agent-zero/issues/1584)

<br/><br/>
# Using Google Vertex

Agent Zero currently does not have Vertex support buit in to its known models.  It is possible to get it working.

Modify /a0/conf/model_providers.yaml to add:

```
  vertex:
    name: Google Vertex API
    litellm_provider: vertex_ai
```

Then in the model dialog pick "Google Vertex API" as the Provider, fill in the model name, and nothing else.

LiteLLM recognizes vertex_ai and knows to pick up the following env vars which can be passed in via Docker:

      - GOOGLE_APPLICATION_CREDENTIALS=<path to .json credentials>
      - VERTEXAI_LOCATION=<vertex location>
      - VERTEXAI_PROJECT=<vertex project>

This works with any models except those that require global location. The reason global doesn't work is that wasn't added until a later version of LiteLLM.  A0 is currently using an older version.  This affects mainly the 3.1 preview models.  Older models work fine with this config.

There is a workaround for global until A0 uses a newer version of LiteLLM. After picking Google Vertex API in the settings, give it a custom URL like this:

```
https://aiplatform.googleapis.com/v1/projects/<vertexproject>/locations/global/publishers/google/models/gemini-3.1-pro-preview
```

There is a pending ticket requesting Vertex support:  [https://github.com/agent0ai/agent-zero/issues/1599](https://github.com/agent0ai/agent-zero/issues/1599)
