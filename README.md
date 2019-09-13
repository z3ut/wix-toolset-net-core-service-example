# WiX Toolset .NET Core service example

WiX toolset .NET Core service installation example

## Description

Creation of .NET Core 2.2 windows service msi installation file with WiX 3.11.

## Steps to setup installation project

* Be sure that your project can be installed and started as service with sc.exe

* In service project add runtime win10-x64 in .csproj

```
<PropertyGroup>
  ...
  <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
</PropertyGroup>
```

* Install WiX with VS extension

* Add new "Setup Project for WiX v3" project in solution

* Add reference to service project in new WiX project: right click on WiX project -> Add -> Reference -> Tab Projects -> Select and add your service project.

* Edit .wixproj file in WiX project to add Heat copying

```
<Target Name="BeforeBuild">
  <Exec Command="dotnet publish ..\WiXToolsetNetCoreService\WiXToolsetNetCoreService.csproj -c $(Configuration) -r win10-x64" />
  <PropertyGroup>
    <LinkerBaseInputPaths>..\WiXToolsetNetCoreService\bin\$(Configuration)\netcoreapp2.2\win10-x64\publish</LinkerBaseInputPaths>
    <DefineConstants>BasePath=..\WiXToolsetNetCoreService\bin\$(Configuration)\netcoreapp2.2\win10-x64\publish</DefineConstants>
  </PropertyGroup>
  <HeatDirectory OutputFile="WiXToolsetNetCoreService.wxs" DirectoryRefId="INSTALLFOLDER" ComponentGroupName="WiXToolsetNetCoreServiceProject"
    SuppressCom="true" Directory="..\WiXToolsetNetCoreService\bin\$(Configuration)\netcoreapp2.2\win10-x64\publish" SuppressFragments="true"
    SuppressRegistry="true" SuppressRootDirectory="true" AutoGenerateGuids="false" GenerateGuidsNow="true" ToolPath="$(WixToolPath)" PreprocessorVariable="var.BasePath" />
</Target>
```

* Fill Manufacturer attribute for Product in Product.wxs in WiX project

* Build WiX project

* Add WiXToolsetNetCoreService.wxs (Heat-henerated file references) to WiX Project in Visual Studio (Add existing item)

* Add reference to WiXToolsetNetCoreService.wxs in Product.wxs

```
<Feature Id="ProductFeature" Title="SetupProject" Level="1">
  <!-- heat references -->
  <ComponentGroupRef Id="WiXToolsetNetCoreServiceProject" />

  <ComponentGroupRef Id="ProductComponents" />
</Feature>
```

* Add service registration in Product.wxs

```
<Fragment>
  <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">

    <!-- service registration -->
    <Component Id="ProductComponent" >
      <File Id="JobServiceEXE"
            Name="WiXToolsetNetCoreService.exe"
            DiskId="1"
            Source="$(var.WiXToolsetNetCoreService.TargetDir)\WiXToolsetNetCoreService.exe"
            Vital="yes"
            KeyPath="yes" />

      <ServiceInstall Id="ServiceInstaller"
                      Type="ownProcess"
                      Vital="yes"
                      Name="WiXToolsetNetCoreService"
                      DisplayName="WiX Toolset .NET Core Service"
                      Description="Example for installing .NET Core service with WiX toolset"
                      Start="auto"
                      Account="LocalSystem"
                      ErrorControl="normal" />

      <ServiceControl Id="StartService"
                      Name="WiXToolsetNetCoreService"
                      Start="install"
                      Wait="no" />
      <ServiceControl Id="StopService"
                      Name="WiXToolsetNetCoreService"
                      Stop="uninstall"
                      Remove="uninstall"
                      Wait="yes" />
    </Component>

  </ComponentGroup>
</Fragment>
```

* Open WiX project properties -> Tool Settings and add ICE30 in field "Suppress specific ICE validation" for all configurations (Debug\Release). This step suppress error about two references to .exe file by Heat config and service installer.

After this you can try to build WiX project and run .msi installation file. The service should be installed in ProgramFilesFolder (c:\Program Files (x86)) and registered in Services.
