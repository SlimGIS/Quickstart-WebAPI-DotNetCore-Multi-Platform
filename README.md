# Quickstart guide to build Web Map APP with ASP.NET Core on macOS and Linux

In the [previous thread](https://slimgis.com/documents/getting-started-webapi-dnc-win), we went over the entire flow of building a web appliction with ASP.NET Core on Visual Studio Code which is a light-weight and cross-platform IDE provided by Microsoft. We also love Visual Studio IDE, but you know, it is kind of heavy-weight IDE to build such simple demo. So this time, we will make challenge again to build a web app with our WebAPI for AspNetCore and Visual Studio Code on macOS and Linux.

Our sample is build with .Net Core 1.1 with SDK RC4 build. Please make sure we have in the same line. Please refer [this article](https://github.com/dotnet/core/blob/master/release-notes/download-archives/rc4-download.md) to get it ready.

## Create project

- After isntalled, `command + t` and type 'terminal' to open termial window.
- Get the version by typing: `dotnet --version`. If it prints `1.0.0-rc4-004771`, it means we have installed the required SDK successfully.
- Follow the commands below to create the webapi project with the project template.

```
cd [YOUR PROJECT FOLDER]
mkdir HelloMap
cd HelloMap
dotnet new webapi
code .
```

The last command means launch VS Code IDE with current folder opened.

## Add dependency packages and referencies

The DotNetCore doesn't support graphics, so we need a 3rd party graphics library [CoreCompat.System.Drawing](https://github.com/CoreCompat/CoreCompat) which is still in beta. I'm sure Microsoft will provide the graphics support in the near future. Because the DotNetCore is so charming. Let's get back to our topic.

- Open *HelloMap.csproj* in VS Code.
- Add the following as package reference for the 3rd party graphics.

*Add this for macOS:*

```
<PackageReference Include="CoreCompat.System.Drawing">
    <Version>1.0.0-beta006</Version>
</PackageReference>    
<PackageReference Include="runtime.osx.10.10-x64.CoreCompat.System.Drawing">
    <Version>1.0.1-beta004</Version>
</PackageReference>
```

*Add this for Linux*

```
<PackageReference Include="CoreCompat.System.Drawing">
    <Version>1.0.0-beta006</Version>
</PackageReference>    
<PackageReference Include="runtime.linux-x64.CoreCompat.System.Drawing">
    <Version>1.0.0-beta009</Version>
</PackageReference>
```

- Create a new folder names *Dependencies* and copy the following two SlimGIS MapKit WebAPI assemblies in it.

    - SGMapKit.Core.dll
    - SGMapKit.WebApi.dll
    
- Add the following to add SlimGIS MapKit WebAPI as reference.

```
<ItemGroup>
    <Reference Include="SGMapKit.WebApi">
        <HintPath>Dependencies\SGMapKit.WebApi.dll</HintPath>
    </Reference>
    <Reference Include="SGMapKit.Core">
        <HintPath>Dependencies\SGMapKit.Core.dll</HintPath>
    </Reference>
</ItemGroup>
```

- `ctrl + \`` to open VS Code's termial panel.
- Type `dotnet restore` to restore the 3rd party dependency.

After it restores complete, our project config part is ready.

## Prepare data and create a new action method

- Create a folder names *AppData*.
- Copy the data from our project to yours.
- Open *Controllers/ValuesController.cs*.
- Create a new method `GetXyzTile` at the bottom.

```
using SlimGis.MapKit.Geometries;
using SlimGis.MapKit.Layers;
using SlimGis.MapKit.Symbologies;
using SlimGis.MapKit.Utilities;
using SlimGis.MapKit.WebAPI;
```

```
[HttpGet]
[Route("{z}/{x}/{y}")]
public IActionResult GetXyzTile(int z, int x, int y)
{
    ShapefileLayer countriesLayer = new ShapefileLayer("AppData/countries-900913.shp");
    countriesLayer.Styles.Add(new FillStyle(GeoColor.FromHtml("#AAFFDF3E"), GeoColors.White));

    MapModel mapModel = new MapModel(GeoUnit.Meter);
    mapModel.Layers.Add(countriesLayer);

    return new XyzTileResult(mapModel, x, y, z);
}
```

We are almost done now.
- Open the terminal again and type following command. 
```
dotnet build
dotnet run
```
- If the terminal prints this, that means our server is launched.

![quickstart-webapi-macos-server-launched](http://i1.piimg.com/567571/1b8995190881dc23.png)
 
- Open web browser and access uri: [http://localhost:5000/api/values/0/0/0](http://localhost:5000/api/values/0/0/0); an image will the countries data will be rendered.
- Go back to VS Code and create an *index.html* file in the root folder. Fill the file with:

```
<!DOCTYPE html>
<html>
<head>
    <title>Quickstart Guide</title>
    <meta charset="utf-8" />
    <link href="https://cdnjs.cloudflare.com/ajax/libs/ol3/3.5.0/ol.css" rel="stylesheet" />
    <style>
        #mapDiv {
            width: 400px;
            height: 400px;
        }
    </style>
</head>
<body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ol3/3.5.0/ol.js" type="text/javascript"></script>
    <div id="mapDiv" />

    <script type="text/javascript">
        let map = new ol.Map({
            layers: [
                new ol.layer.Tile({ source: new ol.source.OSM() }),
                new ol.layer.Tile({
                    source: new ol.source.XYZ({ url: 'http://localhost:5000/api/values/{z}/{x}/{y}' })
                })
            ],
            target: 'mapDiv',
            view: new ol.View({ center: [0, 0], zoom: 2 })
        });
    </script>
</body>
</html>
```

- Double click the index.html, you will see:

![quickstart-webapi-macos-final](http://i1.piimg.com/567571/e28df7cafed88916.png)

![quickstart-webapi-linux-final](http://p1.bpimg.com/567571/dada08f63fd42bdd.png)

Now we have made our first web api running with ASP.NET Core framework. It is so compatible with the project we have done on Windows. 

Happy Mapping! 😃

## Related Resources
- [Source code](https://github.com/SlimGIS/Quickstart-WebAPI-DotNetCore-Multi-Platform)
- [Quickstarted guide on Windows](https://slimgis.com/documents/getting-started-webapi-dnc-win)
- [Map Kit WebAPI for ASP.NET Core introduction](https://slimgis.com/products/webapi-dnc)
- [Map Kit WebAPI introduction](https://slimgis.com/products/webapi)
