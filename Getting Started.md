# 1. GDAL Installation

GDAL can be installed and run on Windows, OSX and Linux and there are different approaches based on your operating system. There is a list [here](https://gdal.org/download.html) that give a good list of options. 

## Installation

In the GDAL docs it recommends to use Conda to install GDAL however one of the alternatives that is not listed is [GIS Internals](https://www.gisinternals.com/) which provide a number of options including MSI installers for Windows for both the stable and development release.

For this workshop we will use stable version and this can be download using the instructions at GDAL site https://gdal.org/download.html. 

| **Windows:** | **`1`. Using network installer at OSGeo4W or  `2.` using conda install -c  conda-forge gdal** |
| ------------ | ------------------------------------------------------------ |
| **Mac:**     | http://www.kyngchaos.com/software/frameworks/    echo 'export PATH=/Library/Frameworks/GDAL.framework/Programs:$PATH' >> ~/.bash_profile source ~/.bash_profile |
| **Ubuntu:**  | sudo  apt-get install gdal-bin                               |

# 2. Getting Started

---
This workshop section will introduce some of the main GDAL utilities that you will use in the rest of the workshop. Make sure you have a working GDAL by running

```gdalinfo --version```

From the command line (terminal, PowerShell or command prompt) and making sure it shows the version you expect.

A list of Utilities/Programs is found [here](https://gdal.org/programs/index.html) and they all have a different purpose and listed below are the most common.

1. __ogr2ogr__ - for Vector spatial data this is the program that you will use to convert from one format to another, or changes it projection or query it directly using SQL
2. __gdal_translate__ - this is the equivalent of ogr2ogr for Raster data. You can use this to do conversions from GeoTiff to ECW, change the compression or color palette.
3. __gdalwarp__ - this is used to change the projection of a Raster file, for example from British National Grid to Web Mercator
4. __gdalbuiltvrt__ - this program builds a Virtual Dataset of many files of the same type, think image catalogue for many imagery files.
5. __gdaladdo__ - this adds internal or external overviews to Raster files which makes viewing these significantly faster at different scales to their native resolution

All of the above programs use and expect different inputs/outputs and parameters and so we will investigate each of them in the other sections.

In this section we get use to using the command line and try out the program 

__gdalinfo/ogrinfo__ - both of these are useful for finding out the metadata of the file. It will give you the format, number of features, projection, bounding box etc

Open a terminal/PowerShell/command prompt window in the data directory of the workshop and run the following command

```gdalinfo --formats```

This will return a list of all the raster spatial formats that are supported in this version of GDAL

![gdalinfo.png](https://cdn-std.droplr.net/files/acc_498240/2pKX2l)

In the list you will see the shorthand name of the supported formats and whether it can read or write that format and then the full format name. It is important to know the shorthand name as this is option used. For example GTiff is GeoTiff.

## 2.1 GDALINFO
---
Now lets use gdalinfo on one of our files to find out the metadata on the file.

Open the command prompt window in the Data/Raster folder where there are a number of images of Columbia at 100 meters resolution are saved. 

We will use __gdalinfo__ to find out more about one of these raster, so lets run

```gdalinfo col_viirs_100m_2016.tif```

And we should get an output like this

![gdal_info.png](https://cdn-std.droplr.net/files/acc_498240/YkI6hB)



The output gives us a lot of information about this file including its coordinate system (if it has one), the pixel size, origin, metadata, the compression used on the file and the color ramp.

In the [docs](https://gdal.org/programs/gdalinfo.html) it lists a number of different parameters that can be used with the gdalinfo command to return different information than the default output. So lets try a few out

JSON Output

```gdalinfo -json col_viirs_100m_2016.tif``` 

This outputs a JSON object which can often be easier to parse.




## 2.2 OGRINFO
---
We can use similar commands on Vector datasets using __ogrinfo__. However it does have a different list of parameters so it is worth looking at the [docs](https://gdal.org/programs/ogrinfo.html#ogrinfo)

so open a new command prompt window in Data/Vector/Shapefile.

1. Default output

```ogrinfo gadm36_COL_1.shp```

2. Summary Output

```ogrinfo -so gadm36_COL_1.shp```

You will notice that it really doesn't give us too much info except the name of the file being opened and what driver is being used to open it. Lets expand on the previous command and provide the layer name as well.

```ogrinfo -so gadm36_COL_1.shp gadm36_COL_1```

![ogrinfo -so.png](https://cdn-std.droplr.net/files/acc_498240/XDUtwS)

You can now see that GDAL gives us lots of really useful information about our file. Type of geometry, feature count, extent, projection and attributes.

If you want to see all the feature information you can run the following command

```ogrinfo -al gadm36_COL_1.shp```

You will see the print out of all the coordinates. So best to avoid this on a massive file.

You can get specific information about a single feature using the __-where__ parameter, so for example

```ogrinfo -so -where GID_1='COL.1_1' gadm36_COL_1.shp gadm36_COL_1```

And if you wanted the full information on that feature you would switch from the __-so__ to the __-al__ parameter.

# 3. Vector Data

---

This part concentrates on using GDAL/OGR to convert, process and manipulate various different vector datasets. The most command line tool used for this is [ogr2ogr](https://gdal.org/programs/ogr2ogr.html#ogr2ogr). By using the variety of parameters listed in the documentation OGR2OGR can be used to do __anything__ with vector data. 

From the docs ogr2ogr is described as

> ogr2ogr can be used to convert simple features data between file formats. It can also perform various operations during the process, such as spatial or attribute selection, reducing the set of attributes, setting the output coordinate system or even reprojecting the features during translation.

So let's crack on and give it a try.

## 3.1 Basic Translation

---

We will start with a very basic command that translate from one spatial format to another, in this case ESRI Shapefile to GeoPackage.

Open a command prompt in the Data/Vector/Shapefile folder and run..

```ogr2ogr -f GPKG gadm36_COL_1.gpkg gadm36_COL_1.shp```

GDAL will run and create our new GeoPackage, open this file in QGIS.

![QGIS GPKG.png](https://cdn-std.droplr.net/files/acc_498240/IatyoM)



Not very exciting translation but it introduces several key principles.

- We are invoking ogr2ogr (which is found in our PATH so do not need to give the full path to the exe file)
- We set the output file format using -f
- Output filename
- Input filname

Lets download dataset for India from GDAM https://gadm.org/download_country_v3.html

![Screenshot at May 12th 2020 - 1.29.12 pm.png](https://cdn-std.droplr.net/files/acc_498240/cKxCeI)

## 3.2 Clip vector file

```ogr2ogr -clipdst 70 10 78 28 IND_clip1.shp gadm36_IND_3.shp```
```ogr2ogr -clipdst 78 10 86 28 IND_clip2.shp gadm36_IND_3.shp```

![Screenshot at May 12th 2020 - 1.34.03 pm.png](https://cdn-std.droplr.net/files/acc_498240/jN1pD0)

Inside python you can use** **ogrmerge** **to merge several vector datasets into a single one

## 3.3 Setting and using Projection Transformations

---

Sometimes we can translate a file and it might not have a spatial reference file (eg for ESRI Shapefile a .proj file) and therefore applications like QGIS have no idea where this dataset lives. So on opening the file you might see the prompt to ask you

Within ogr2ogr there are three parameters we can use to set the coordinate reference system or even translate from one CRS to another.

- __-a_srs__ is used to __ASSSIGN__ (hence the a) a CRS with the file.
- __-s_srs__ is used to set the __SOURCE__ (hence the s) CRS of the file
- __-t_srs__ is used to set the __TARGET__ (hence the t) CRS to transform to

We can see this in action by converting India ESRI Shapefile 

Projecting from GCS_WGS_1984 to EPSG:32644 (UTM 44N) .

```ogr2ogr -t_srs EPSG:32644 IND_prj.shp gadm36_IND_3.shp```


# 4. Raster Data


```gdalinfo lights.tif```

```gdal_merge –o mymerge.tif t27elu.dem t28elu.dem –ps 20 20```

Create contour lines

```gdal_contour mymerge.tif mycontours.shp –i 20```
Hillshade

```gdaldem hillshade mymerge.tif hillshade_30.tif –alt 45```
Slope

```gdaldem slope mymerge.tif slope.tif```

raster calculator

```gdal_calc --calc=“A*(A>80)” -A slope.tif --outfile=slopebin.tif```