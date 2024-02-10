---
layout: post
title: Change Performance Monitor Process Name Format
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [perfmon]
comments: true
---

The default behavior of Performance Monitor is to display performance counters on a per-application bases. For example, if we have two instances for a "myapp.exe", we will see "myapp" and "myapp#1".

Like so:

![](/assets/img/articles/changePCFormat/img1.png)


This is the default behavior.

However, we can change the format by modifying a registry key.

We need to create a registry key called "ProcessNameFormat" with a value of "2".

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PerfProc\Performance


"ProcessNameFormat"=dword:00000002
```

![](/assets/img/articles/changePCFormat/img2.png)

**Restart Performance Monitor.**

![](/assets/img/articles/changePCFormat/img3.png)


Same behavior can also be apply to CLR counters data by creating key at a different path.

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\.NETFramework\Performance


"ProcessNameFormat"=dword:00000002
```
**Restart Performance Monitor.**

![](/assets/img/articles/changePCFormat/img4.png)

## Resources

* [Registry change for perfmon and PID data](https://asp-blogs.azurewebsites.net/owscott/registry-change-for-perfmon-and-pid-data)

* [Performance Counters and In-Process Side-By-Side Applications](https://learn.microsoft.com/en-us/dotnet/framework/debug-trace-profile/performance-counters-and-in-process-side-by-side-applications)