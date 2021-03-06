# How to create and run MPI projects in CLion

1. Download and install MPI from [Microsoft Website](https://www.microsoft.com/en-us/download/details.aspx?id=100593). You need to download both `msmpisetup.exe` and `msmpisdk.msi`.

> Note that you shouldn't have spacec or cyrillic characters in path to MPI installation.

2. Open Windows Terminal and try running `mpiexec`:

```bash
Microsoft MPI Startup Program [Version 10.1.12498.18]

...
```

If the `mpiexec` command isn't recognised by your system try restarting your PC and check if MPI system variables are installed correctly:

| Name | Value |
| --- | --- |
| MSMPI_BENCHMARKS | `path_to_mpi\Benchmarks\` |
| MSMPI_BIN | `path_to_mpi\Bin\` |
| MSMPI_INC | `path_to_mpi\SDK\Include\` |
| MSMPI_LIB32 | `path_to_mpi\SDK\Lib\x86\` |
| MSMPI_LIB64 | `path_to_mpi\SDK\Lib\x64\` |

And `path_to_mpi\Bin\` path is added to `Path` variable.

> To view system variables open Windows Search -> Input "Variables" -> Click on "Edit the system environment variables" -> Click on "Environemnt Variables on "System Properties" Window.

> `path_to_mpi` is a path to your Microsoft MPI installation.


3. Open CLion and create new `C++ Executable` Project.

4. Open `CMakeLists.txt` where you should see something like this: (`MY_PROJECT` is project name)

```cmake
cmake_minimum_required(VERSION 3.20)
project(MY_PROJECT)

set(CMAKE_CXX_STANDARD 14)

add_executable(MY_PROJECT main.cpp)

```

5. Add MPI Package and MPI Library to project

```cmake
cmake_minimum_required(VERSION 3.20)
project (MY_PROJECT)

set(CMAKE_CXX_STANDARD 14)

# Add MPI Package to Project
find_package(MPI REQUIRED)

add_executable(MY_PROJECT main.cpp)
# Add libraries for code completion and compiling
target_link_libraries(MY_PROJECT PUBLIC MPI::MPI_CXX)
```

6. Add following run configurations:

**1) CMake Application**

| Property | Value |
| --- | --- |
| Name | `Compile` |
| Tagret | `MY_PROJECT` |
| Executable | `MY_PROJECT` |
| Environment Variables | `MPI_CXX_INCLUDE_PATH=path_to_mpi\SDK\Include\;MPI_CXX_LIBRARIES=path_to_mpi\SDK\Lib\x64\;MPI_CXX_BIN_PATH=path_to_mpi\Bin\` |

> IMPORTANT! Replace `path_to_mpi` string with a path to your Microsoft MPI intallation!

**2) Shell Script**

| Property | Value |
| --- | --- |
| Name | `2 Cores` |
| Execute | `Script text` |
| Script text | `mpiexec -n 2 ./cmake-build-debug/MY_PROJECT` |
| Working directory | `project_directory/MY_PROJECT` |
| Execute in the termial | `true` |

> Add more shell scripts for number of cores you would like to use while executing your project. Replace number `2` in `Name` and `Script text` properties with number of cores you would like to use.

> `MY_PROJECT` is a name of `.exe` file in `./cmake-build-debug` folder.

7. Add folling code to `main.cpp`:

```cpp
#include <mpi.h>
#include <cstdio>

int main(int argc, char** argv) {
	// Initialize the MPI environment
	MPI_Init(NULL, NULL);

	// Get the number of processes
	int world_size;
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);

	// Get the rank of the process
	int world_rank;
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

	// Get the name of the processor
	char processor_name[MPI_MAX_PROCESSOR_NAME];
	int name_len;
	MPI_Get_processor_name(processor_name, &name_len);

	// Print off a hello world message
	printf("Hello world from processor %s, rank %d out of %d processors\n",
		   processor_name, world_rank, world_size);

	// Finalize the MPI environment.
	MPI_Finalize();
}
```

8. Setup CMake "Release" configuration. Go to `Settings` -> `Build, Execution, Deployment` -> `CMake`. Click on "Plus" sign and a new "Release" configuration should be created. Click on "OK" or "Apply".

9. Select "Release" under "Configurations" dropdown.

10. Run `Compile` configuration before executing any of `X Cores` configurations.

> If you are getting an error "Cannot start process, the working directory '../cmake-build-release' does not exist" then RMC on your project folder and select "Reload CMake Project".

