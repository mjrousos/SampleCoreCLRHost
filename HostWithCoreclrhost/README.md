Sample .NET Core Host (CoreClrHost.h)
=====================================

This repository contains a simple .NET Core host using hosting APIs from [CoreClrHost.h](https://github.com/dotnet/coreclr/blob/master/src/coreclr/hosts/inc/coreclrhost.h). The sample host loads and starts the .NET Core CLR (including starting the default AppDomain), loads managed code, calls into a managed method, and provides a function pointer for the managed code to call back into the host.

This host is not meant to cover all possible hosting scenarios. Instead, it is meant to demonstrate mainline use cases. The host can be built either for Windows or Linux using the simple build scripts (build.bat and build.sh).

Host to Host .NET Core
----------------------

The host in this repository walks through the following steps necessary to host the CoreCLR using coreclrhost.h APIs. The steps here match comments in [SampleHost.cpp](src/SampleHost.cpp).

1. Load the .NET Core CoreCLR library (coreclr.dll on Windows, libcoreclr.so on Linux/Mac). This library contains the functions necessary to start the .NET Core runtime. In some hosting cases, it will be necessary to probe around to locate coreclr.dll/libcoreclr.so. In other cases (as with this sample), the library is just assumed to be next to the host. Functions like `LoadLibraryEx` on Windows or `dlopen` on Linux can be used for loading the library.
2. Get function pointers for the hosting APIs you intend to use (using `GetProcAddress` or `dlsym`). Useful coreclrhost.h functions you may need include:
    1. `coreclr_initialize`: Starts the .NET Core runtime and default domain
    1. `coreclr_execute_assembly`: Executes a managed assembly
    1. `coreclr_create_delegate`: Creates a function pointer to a managed method
    1. `coreclr_shutdown`: Unloads AppDomains and shuts down the .NET Core runtime
    1. `coreclr_shutdown_2`: Like `coreclr_shutdown`, but also retrieves the managed code's exit code
3. Prepare to start the runtime by readying AppDomain properties. There are a number of important AppDomain properties which must be provided at AppDomain-creation time.
    1. **`APPBASE`** is the base path of the application from which the exe and other assemblies will be loaded. 
    1. **`TRUSTED_PLATFORM_ASSEMBLIES`** lists the known-Framework assemblies that can be loaded in the AppDomain. The loader will prefer assemblies on this list to those found through further probing and these assemblies are assumed to be trusted by the host and granted full trust even in sandboxed AppDomains. Typically, the TPA list (as this property is called) includes all assemblies next to CoreCLR.dll.
    1. **`APP_PATHS`** lists directories for the loader to probe in for needed assemblies which aren't found on the TPA list. These assemblies are loaded as partially trusted in sandboxed AppDomains. Typically, the path of the executing assembly is included in this list.
    1. **`APP_NI_PATHS`** is similar to APP_PATHS but lists paths to probe in for native images (created via coregen/crossgen).
    1. **`NATIVE_DLL_SEARCH_DIRECTORIES`** is similar to APP_PATHS, but instead of listing where to probe for managed dependencies, it lists directories to be searched for native dependencies that managed code may PInvoke into.
    1. **`PLATFORM_RESOURCE_ROOTS`** lists paths to probe for satellite resource assemblies. Resource assemblies will be looked for in culture-appropriate sub-directories of items in this list.
4. Start the .NET Core runtime and create the default AppDomain by calling `coreclr_initialize`. The AppDomain properties created earlier are provided here, along with a name for the default domain and a host handle and AppDomain ID are returned.
5. Run managed code. This can be done in one of two ways:
      1. Use `coreclr_execute_assembly` to launch a managed executable. This takes an assembly path and arrray of arguments as input parameters. It loads the assembly at that path and invokes its main method. The main method's return code is returned.
      1. Use `coreclr_create_delegate` to create a function pointer to a static managed method. The API takes the assembly name, namespace-qualified type name, and method name to be called and returns a function pointer to the managed method.
6. Cleanup by unloading the AppDomain and stopping the CoreCLR using `coreclr_shutdown` or `coreclr_shutdown_2`. Be sure to also free the CoreCLR library (`FreeLibrary` or `dlclose`).

Other Resources
---------------

More full-featured hosts can be found in the [dotnet/coreclr](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts) repository. The [Unix CoreRun host](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts/unixcorerun) (which uses code from [unixcoreruncommon](https://github.com/dotnet/coreclr/tree/master/src/coreclr/hosts/unixcoreruncommon)), in particular, is a good general-purpose host to study after reading through this sample.
