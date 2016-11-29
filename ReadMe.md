Sample CoreCLR Host
===================

This repository contains a simple, bare-bones CoreCLR host used to demonstrate the steps necessary to start the .NET Core common language runtime, create a default AppDomain, and execute a managed assembly in it.

This host is not meant to cover all possible hosting scenarios. Instead, it is made with the intention of demonstrating mainline hosting scenarios in an easy-to-follow way for learning purposes.

How to Host the CoreCLR
-----------------------

The host in this repository walks through the following steps necessary to host the CoreCLR. The steps here match comments in host.cpp.

1. Determine which managed assets to load, as appropriate. For most hosts, this is a matter of command line parsing.
2. Find and load CoreCLR.dll, which the host will use to start the runtime. The .NET Core runtime will be most easily found if it is in a well-known location relative to the host or if it can be expected in the usual .NET Core install locations. In other scenarios, users may specify which CoreCLR.dll to use through command line parameters or environment variables.
3. Get an instance of the `ICLRRuntimeHost2` interface from CoreCLR.dll by calling `GetCLRRuntimeHost`.
	1. In Unix hosts, the `coreclr_initialize` API may be used to both initialize the runtime and the default AppDomain.
4. Start the CoreCLR (via `ICLRRuntimeHost::Start`) after setting startup flags (to control such settings as which GC to use, load optimizations, etc.).
5.  Prepare to create an AppDomain by readying AppDomain properties and flags. There are a number of important AppDomain properties which must be provided at AppDomain-creation time.
	1. **`TRUSTED_PLATFORM_ASSEMBLIES`** lists the known-Framework assemblies that can be loaded in the AppDomain. The loader will prefer assemblies on this list to those found through further probing and these assemblies are assumed to be trusted by the host and granted full trust even in sandboxed AppDomains. Typically, the TPA list (as this property is called) includes all assemblies next to CoreCLR.dll.
	2. **`APP_PATHS`** lists directories for the loader to probe in for needed assemblies which aren't found on the TPA list. These assemblies are loaded as partially trusted in sandboxed AppDomains. Typically, the path of the executing assembly is included in this list.
	3. **`APP_NI_PATHS`** is similar to APP_PATHS but lists paths to probe in for native images (created via coregen/crossgen).
	4. **`NATIVE_DLL_SEARCH_DIRECTORIES`** is similar to APP_PATHS, but instead of listing where to probe for managed dependencies, it lists directories to be searched for native dependencies that managed code may PInvoke into.
	5. **`PLATFORM_RESOURCE_ROOTS`** lists paths to probe for satellite resource assemblies. Resource assemblies will be looked for in culture-appropriate sub-directories of items in this list.
	6. **`AppDomainCompatSwitch`** indicates what .NET Framework behavior (old, Silverlight/Phone behavior, or more modern .NET Core behavior) should be used in Frameworks which don't explicitly include a Target Framework Moniker. 
	7.  AppDomain flags are also provided to control security, interop, and other behaviors in the AppDomain. 
6. Create the default AppDomain with a call to `ICLRRuntimeHost2::CreateAppDomainWithManager`. Note that an AppDomain manager may optionally be provided here to control some aspects of AppDomain behavior and to provide a convenient entry-point for the host in scenarios in which the user doesn't provide one.
	1. Again, Unix hosts will likely use `coreclr_initialize` to accomplish this.
7. Run managed code using `ICLRRuntimeHost2::ExecuteAssembly`. This function takes path to managed assembly and executes its entry point.
	1. Alternatively, if you want to run managed code that isn't an entrypoint, `ICLRRuntimeHost2::CreateDelegate` can create a function pointer to arbitrary static managed methods.
	2. In Unix hosts, managed code is most easily launched with the `coreclr_execute_assembly` API. 
8. Cleanup by unloading the AppDomain, and stopping/releasing the CoreCLR.
	1. On Unix, use `coreclr_shutdown`.

Other Resources
---------------

More full-featured hosts can be found in the [dotnet/coreclr](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts) repository. The [CoreRun host](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts/corerun), in particular, is a good general-purpose host to study after reading through this sample. CoreRun also has a [Unix port](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts/unixcorerun). 
