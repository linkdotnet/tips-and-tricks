## Floating version
NuGet gives the option to use wildcards instead of concrete versions. This gives the ability to use always the latest (major/minor/patch) version of a certain package. 

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <ItemGroup>
    <PackageReference Include="Take.Latest.Minor.Of.Version.3" Version="3.*" />
    <PackageReference Include="Take.Latest.Patch.Of.Version.3.2" Version="3.2.*" />
  </ItemGroup>
</Project>
```

This can be especially helpful when working with preview version of dotnet itself. If you always want to have the latest preview version of let's say the OpenIdConnect-package and Entity Framework core, you could do the following:
```xml
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.Authentication.OpenIdConnect" Version="7.0.0-*" />
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="7.0.0-*" />
</ItemGroup>
```