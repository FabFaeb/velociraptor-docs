---
title: Linux.RHEL.Packages
hidden: true
tags: [Client Artifact]
---

Parse packages installed from dnf or yum


<pre><code class="language-yaml">
name: Linux.RHEL.Packages
description: |
  Parse packages installed from dnf or yum

parameters:
  - name: DNFGrokExpression
    description: A Grok expression to parse the output from `dns list installed`
    default: "%{DATA:Package}%{SPACE} %{DATA:Version}%{SPACE} %{GREEDYDATA:Repository}"

sources:
  - precondition: |
      SELECT OS From info() where OS = 'linux'

    query: |
        LET exec = SELECT * FROM foreach(row={
          SELECT grok(grok=DNFGrokExpression, data=Stdout) AS Parsed
          FROM execve(argv=cmd, sep="\n")
          WHERE count() &gt; 2
        }, column="Parsed")
        
        SELECT * FROM switch(
            a=exec(cmd=["dnf", "--quiet", "list", "installed"]),
            b=exec(cmd=["yum", "--quiet", "list", "installed"]),
            c={SELECT log(level="ERROR",message="Could not retrieve Package Information") FROM scope()}
            )

</code></pre>

