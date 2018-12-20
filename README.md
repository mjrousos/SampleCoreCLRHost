Sample .NET Core Hosts
======================

This repository contains sample code demonstrating how to host managed .NET Core code in a native process. These hosts bypass the usual `dotnet` host and launch managed code directly.

There are two samples demonstrating two different hosting interfaces for .NET Core.

1. The [HostWithMscoree](HostWithMscoree) folder contains a sample host using the `ICLRRuntimeHost2` interface from msocree.h to host .NET Core. This is the interface that was originally used to host .NET Core and it still has a few more APIs than the CoreClrHost alternative (including unsupported features like creating multiple AppDomains).
1. The [HostWithCoreclrHost](HostWithCoreclrHost) folder demonstrates how to host the .NET Core runtime using the newer CoreCLRHost.h API. This API is the preferred method of hosting .NET Core and is a bit simpler to use than mscoree.h.

These hosts are both small and bypass a lot of complexity (probing for assemblies in multiple locations, thorough error checking, etc.) that a real host would have. Hopefully because by remaining simple, though, they will be useful for demonstrating the core concepts of hosting managed .NET Core code in a native process.