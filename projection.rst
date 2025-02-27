Projection
==========

Create Projection
-----------------

.. code-block:: python


	from osgeo import osr
	spatialRef = osr.SpatialReference()
	spatialRef.ImportFromEPSG(2927) 	# from EPSG

Reproject a Geometry
--------------------

.. code-block:: python

    from osgeo import ogr
    from osgeo import osr

    source = osr.SpatialReference()
    source.ImportFromEPSG(2927)

    target = osr.SpatialReference()
    target.ImportFromEPSG(4326)

    transform = osr.CoordinateTransformation(source, target)

    point = ogr.CreateGeometryFromWkt("POINT (1120351.57 741921.42)")
    point.Transform(transform)

    print point.ExportToWkt()

Get Projection
--------------

.. code-block:: python

	from osgeo import ogr, osr
	driver = ogr.GetDriverByName('ESRI Shapefile')
	dataset = driver.Open(r'c:\data\yourshpfile.shp')
	
	# from Layer
	layer = dataset.GetLayer()
	spatialRef = layer.GetSpatialRef() 
	# from Geometry
	feature = layer.GetNextFeature()
	geom = feature.GetGeometryRef()
	spatialRef = geom.GetSpatialReference() 


Reproject a Layer
-----------------

.. code-block:: python

    from osgeo import ogr, osr
    import os

    driver = ogr.GetDriverByName('ESRI Shapefile')

    # input SpatialReference
    inSpatialRef = osr.SpatialReference()
    inSpatialRef.ImportFromEPSG(2927)

    # output SpatialReference
    outSpatialRef = osr.SpatialReference()
    outSpatialRef.ImportFromEPSG(4326)

    # create the CoordinateTransformation
    coordTrans = osr.CoordinateTransformation(inSpatialRef, outSpatialRef)

    # get the input layer
    inDataSet = driver.Open(r'c:\data\spatial\basemap.shp')
    inLayer = inDataSet.GetLayer()

    # create the output layer
    outputShapefile = r'c:\data\spatial\basemap_4326.shp'
    if os.path.exists(outputShapefile):
        driver.DeleteDataSource(outputShapefile)
    outDataSet = driver.CreateDataSource(outputShapefile)
    outLayer = outDataSet.CreateLayer("basemap_4326", geom_type=ogr.wkbMultiPolygon)

    # add fields
    inLayerDefn = inLayer.GetLayerDefn()
    for i in range(0, inLayerDefn.GetFieldCount()):
        fieldDefn = inLayerDefn.GetFieldDefn(i)
        outLayer.CreateField(fieldDefn)

    # get the output layer's feature definition
    outLayerDefn = outLayer.GetLayerDefn()

    # loop through the input features
    inFeature = inLayer.GetNextFeature()
    while inFeature:
        # get the input geometry
        geom = inFeature.GetGeometryRef()
        # reproject the geometry
        geom.Transform(coordTrans)
        # create a new feature
        outFeature = ogr.Feature(outLayerDefn)
        # set the geometry and attribute
        outFeature.SetGeometry(geom)
        for i in range(0, outLayerDefn.GetFieldCount()):
            outFeature.SetField(outLayerDefn.GetFieldDefn(i).GetNameRef(), inFeature.GetField(i))
        # add the feature to the shapefile
        outLayer.CreateFeature(outFeature)
        # dereference the features and get the next input feature
        outFeature = None
        inFeature = inLayer.GetNextFeature()

    # Save and close the shapefiles
    inDataSet = None
    outDataSet = None

Export Projection
-----------------

.. code-block:: python

	from osgeo import ogr, osr
	driver = ogr.GetDriverByName('ESRI Shapefile')
	dataset = driver.Open(r'c:\data\yourshpfile.shp')
	layer = dataset.GetLayer()
	spatialRef = layer.GetSpatialRef() 
	
	spatialRef.ExportToWkt()
	spatialRef.ExportToPrettyWkt()
	spatialRef.ExportToPCI()
	spatialRef.ExportToUSGS()
	spatialRef.ExportToXML()
	
	
Create an ESRI.prj file
-----------------------

.. code-block:: python

	from osgeo import ogr, osr

	spatialRef = osr.SpatialReference()
	spatialRef.ImportFromEPSG(26912)

	spatialRef.MorphToESRI()
	file = open('yourshpfile.prj', 'w')
	file.write(spatialRef.ExportToWkt())
	file.close()

Create a vector layer with srs definition
-------------------------------------------

.. code-block:: python

    driver = ogr.GetDriverByName("ESRI Shapefile")
    if os.path.isfile(filename): os.unlink(filename)
    
    srs3857 = osr.SpatialReference()
    srs3857.ImportFromEPSG(3857)    
    srs4326 = osr.SpatialReference()
    srs4326.ImportFromEPSG(4326)
    outDataSource = driver.CreateDataSource(filename)
    outLayer = outDataSource.CreateLayer(filename, srs=srs3857, geom_type=ogr.wkbMultiLineString)
    textField = ogr.FieldDefn('SC_9', ogr.OFTString)
    outLayer.CreateField(textField)
    
    outLayer = None
    outDataSource = None
    driver = None

Test for layer projection
--------------------------------------
.. code-block:: python

    from osgeo import ogr, osr
    
    driver = ogr.GetDriverByName('ESRI Shapefile')
    dataset = driver.Open('data.shp')
    layer = dataset.GetLayer()
    
    srs4326 = osr.SpatialReference()
    srs4326.ImportFromEPSG(4326)
    spatialRef = layer.GetSpatialRef()
    if not (spatialRef.IsSame(srs4326)):
        raise OperationException('Invalid vector input. SRS is not EPSG:4326.')        


