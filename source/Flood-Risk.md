---

title: 'Flood Risk Analytics'

---

# Flood Risk Analytics

<div class="date">August 4, 2015</div>

## Flood Risk Assessment

For insurance companies to accurately model flood risk, primarily two sources of information are needed. First is information about flood severity, which can be derived from current and historic water location (rivers, coastlines, lakes), stores (glaciers, snowmelt), and levels (topology). Second is an analysis of vulnerability, which is based on the properties within these regions.

Property insurance firms require the ability to combine such information about natural structures with high geographic resolution. Historic information is of limited value due to limited detail of existing records (in both a spatial and temporal sense) and the rarity of flood events. Additionally climate change causes local change in weather patterns and surface modifications lead to altering water run-off pathways.

Despite a multitude of data sources, there has been a drought in actionable information and as a result flood modeling has been both uncertain and ambiguous. The uncertainty and ambiguity has tremendous financial implications as in the year 2013 alone, water-hazard-related losses amounted to [nearly one trillion USD](http://www.munichre.com/).

For an evidence-based risk modeling to be a reality, it is essential to have access to high-resolution data collected over the entire area of insured properties on a regular basis. With this dynamic information, it would be possible to couple the models for distribution of the flood severity with the vulnerability of the insurance portfolio and arrive transparently at a concrete, traceable value for total loss prediction. The modeled information can further be used in various analytical areas of general insurance firms such as in pricing, reserving, and capital modeling for solvency purposes.

## Next Generation Satellite Imaging with IQAE

In parallel, the latest satellite developments from NASA, Google, Airbus Defense EADS and others provide frequent (weekly) high resolution (10m) imaging of the entire planet. These measurements result in petabytes of data generation.

<div class="half-width-image"><img src="images/flood-risk/fr-001.png"></div>

### Image Query and Analysis Engine

<div class="half-width-image"><img src="images/flood-risk/fr-002.png"></div>

Our tools enable large, complicated satellite imaging datasets to be processed easily and scaled across hundreds of machines.

### Example

#### Locate bodies of water in the image

Here we locate and segment the water for a single day in the following images.

<div class="code">
  SELECT feature FROM ( <br>
    SELECT SEGMENT_WATER(image.blue) FROM SatelliteImages  <br>
    ) WHERE AREA > 1000m^2 AND DATE IS 4-8-2015
</div>

<div class="half-width-image">
  <img src="images/flood-risk/fr-001.png">
  <img src="images/flood-risk/fr-003.png">
</div>

The data can also be processed over months even years worth of old images to characterize the standard changes in water levels.

<div class="code">
  SELECT DATE,AREA FROM ( <br>
    SELECT SEGMENT_WATER(image.blue) FROM SatelliteImages  <br>
    ) WHERE AREA > 1000m^2 AND  <br>
      DATE BETWEEN 4-8-2000 AND 4-8-2015
</div>

<img src="images/flood-risk/fr-004.png">

This information can then be summarized by year to get a better idea of the most extreme cases from a large pool

<div class="code">
  SELECT DATE.YEAR,MIN(AREA),MAX(AREA) FROM ( <br>
    SELECT SEGMENT_WATER(image.blue) FROM SatelliteImages  <br>
    ) WHERE AREA > 1000m^2 AND  <br>
      DATE BETWEEN 4-8-2000 AND 4-8-2015 <br>
      GROUP BY DATE.YEAR <br>
</div>

<img src="images/flood-risk/fr-005.png">

| year | min.area  | max.area  |
| ---  |    ---    |    ---    |
| 2000 | 0.0006822 | 0.0017186 |
| 2001 | 0.0002175 | 0.0017342 |
| 2002 | 0.0003068 | 0.0015424 |
| 2003 | 0.0002740 | 0.0018010 |
| 2004 | 0.0001725 | 0.0018281 |
| 2005 | 0.0001795 | 0.0016319 |
| 2006 | 0.0001851 | 0.0018769 |
| 2007 | 0.0003189 | 0.0019106 |
| 2008 | 0.0007270 | 0.0022661 |
| 2009 | 0.0006729 | 0.0021103 |
| 2010 | 0.0007058 | 0.0020922 |
| 2011 | 0.0006443 | 0.0021316 |
| 2012 | 0.0008001 | 0.0020073 |
| 2013 | 0.0006278 | 0.0021103 |
| 2014 | 0.0008639 | 0.0022816 |
| 2015 | 0.0005949 | 0.0018625 |

#### Identify houses in image

<div class="code">
  SELECT feature FROM ( <br>
    SELECT SEGMENT_BUILDING(image) FROM SatelliteImages  <br>
    ) WHERE SIZE > 100m^2 AND DATE IS 4-8-2014 <br>
</div>

<div class="half-width-image">
  <img src="images/flood-risk/fr-001.png">
  <img src="images/flood-risk/fr-006.png">
</div>

#### Identify the houses distance from the river

<div class="code">SELECT bd.pos,rv.distance FROM Buildings AS bd JOIN Distance(River) AS rv ON bd.pos = rv.pos</div>

<div class="half-width-image">
  <img src="images/flood-risk/fr-007.png">
  <img src="images/flood-risk/fr-006.png">
</div>

<img src="images/flood-risk/fr-008.png">

Or an interactive map for more exact examination:

<iframe height="560px" width="560px" src="http://4quant.com/Flood-Risk/widget_building-distance.html"></iframe>

Furthermore the data can be analyzed using standard tools like Excel, R, or Matlab for further quantifying and modeling of the risk.

<img src="images/flood-risk/fr-009.png">

A more detailed distribution which allows us to see that the larger more expensive buildings (the category furthest on the right) are preferentially located close to the river. This could be a major concern for future risks.

<img src="images/flood-risk/fr-010.png">

## How

There are two interfaces available the *data consumer* and the *data scientist*.

<img src="images/flood-risk/fr-011.png">

* For the **data consumer** There interface is SQL console (JDBC-compatible), which can be examined with ad-hoc queries as shown above. The data can be combined and integrated with other sources using standard SQL operations and the full class of 4Quant’s imaging tools are available as integrated functions.
* For the **data scientist** There is a Scala/Java and Python interface. With this interface new code can be compiled and execute to process the images and other data. Anything from new image processing functions (denoising, feature recognition, optical character recognition) to further analyses (covariance matrix-based identification) can easily be integrated in our high-throughput pipeline.

### Distributing the Analysis

The first question is how the data can be processed. The basic work is done by a simple workflow on top of our Spark Image Layer. This abstracts away the complexities of cloud computing and distributed analysis. You focus only on the core task of image processing.

<img src="images/flood-risk/fr-012.svg" style="width: 40%;">

Beyond a single train, our system scales linearly to multiple datasources and petabytes of information across hundreds of computers to keep the computation real-time.

<img src="images/flood-risk/fr-013.svg">

With cloud-integration and Big Data-based frameworks, even handling an entire satellite network with 100s of satellites continuously collecting data is an easy task without worrying about networks, topology, or fault-tolerance. Below is an example for 30 different map sources where the tasks are seamlessly, evenly divided among 50 different machines.

<img src="images/flood-risk/fr-014.svg">

### Processing the Data

Once the cluster has been comissioned and you have the SparkContext called `sc` (automatically provided in [Databricks Cloud](https://databricks.com/product/databricks) or [Zeppelin](http://zeppelin.incubator.apache.org/), the data can be loaded using the Spark Image Layer. Since for this case we load the images from a private repository stored on Amazon’s S3 block storage (also compatible with local filesystems, Google Compute, IBM SoftLayer)

<div class="code">
  val fullSatImage = sc.readTiledImage\[Double]("s3n://geo-images/esri-satimg/*/*.png",256,256).cache
</div>

Although we execute the command on one machine, the analysis will be distributed over the entire set of cluster resources available to `sc`. To further process the images, we can take advantage of the rich set of functionality built into Spark Image Layer. Watch [our webinar](https://www.youtube.com/watch?v=TA_h0jY7iPc) to learn more.

### Learn More

To find out more about the technical aspects of our solution, check out our presentation at the [Spark Summit](http://4quant.com/spark-east-2015) or watch the [video](https://www.youtube.com/watch?v=ohR_y7HZaHA&index=10&list=PL-x35fyliRwiy50Ud2ltPx8_yA4H34ppJ).
Check out our other demos to see how 4Quant can help you:

* [Counting Cars in Satellite Images](http://4quant.com/countingcarsdemo)
* [Finding buildings and forests in Satellite Images](http://4quant.com/geospatialdemo/)
* [Check train tracks in real time](http://4quant.com/Railway-Check)
* [Track criminals in cars using traffic cameras](http://4quant.com/Pursuing-Criminals/)
* [Count people from drone footage](http://4quant.com/Drone-People-Counting)
