[![Build status](https://ci.appveyor.com/api/projects/status/7rlfl6ur1uny4449/branch/master?svg=true)](https://ci.appveyor.com/project/qbikez/csproj/branch/master)
[![Gitter Chat](https://badges.gitter.im/qbikez/ps-csproj.png)](https://gitter.im/heavymetaldev/Lobby)

## Installation

If you're running Windows 10 or later, install from [PowerShell Gallery](https://www.powershellgallery.com/packages/csproj/):

```powershell
PS> PowerShellGet\Install-Module csproj
```

On Windows 8, you'll neeed to install PackageManagement manually (using [psbootstrap](https://github.com/qbikez/ps-bootstrap)) to be able to use `Install-Module`:

```powershell
PS> ((New-Object System.Net.WebClient).DownloadString('https://bit.ly/psbootstrap')) | out-file "$env:localappdata/ps-bootstrap/bootstrap.ps1"; & "$env:localappdata/ps-bootstrap/bootstrap.ps1"
PS> PowerShellGet\Install-Module csproj
```

## Usage

```powershell
PS> import-module csproj
```

### Create and push nuget for a csproj

```powershell
PS> cd "src/myproject"
PS> Push-Nuget -build -source https://www.nuget.org/api/v2/package
```

This will:
 * find *csproj* file in *src/my.project*
 * update version in *Properties/AssemblyInfo.cs*, according to [Versioning convention](#versioning-conventions) 
 * build it
 * pack nuget, by invoking `nuget pack`
 * push nuget, by invoking `nuget push`
 
Other usefull parameters:
 * `-Stable` - generate a stable nuget version (without suffix)
 * `-IncrementVersion` - incrememt `patch` version component
 * `-Suffux` - manually set version suffix


### Create and push nugets for multiple projects

```powershell
PS> cd "reporoot"    
PS> push-nugets -project "My.Project","Other.Project" -build -source https://www.nuget.org/api/v2/package
```

This will:
 * scan your repo for `*.csproj` files (this is done only once and cached in `.projects.json` file)
 * invoke `push-nuget` with provided parameters for every project given in `-project`

Other usefull parameters:
 * `-allowNoNuspec` - allow pushing of packages without a .nuspec file
 * `-scan` - rescan repo directory for projects

### Convert project references to nuget references


1. Make sure the project you're trying to convert is installed as a nuget:

```powershell
PS> nuget install "Project.To.Convert" -out "packages"
```       

2. Convert all references to that projects:

```powershell
PS> import-module csproj
PS> tonuget "path\to\my\solution.sln" -projectName "Project.To.Convert" -packagesDir "packages"
```

This will:
 * scan all `csproj` files referenced by `solution.sln`
 * replace all project references to `Project.To.Convert` with nuget references to `packages\Project.To.Convert.LatestVersion`
 * remove `Project.To.Convert` from `solution.sln` 


## Versioning conventions <a id="versioning-conventions"></a>

We try to follow [GitVersion](http://gitversion.readthedocs.io/en/latest/examples/) as close as possible, while keeping compatibility with nuget.

The version template looks like this:

    major.minor.patch-{branch-name}{buildno}-{commit hash}

* `{branch-name}` and `{commit hash}` will be filled if project is under version control by Git or Mercurial
* `{buildno}` is incremented each time you call `push-nuget -build`
* whole suffix part is limited to 20 chars: 10 for branch,  3 fro buildno, 6 for commit hash

    
# Contributing

1. Initialize dependencies:

    ```shell
    npm run init
    ```

2. Test:

    ```powershell
    npm test
    ```