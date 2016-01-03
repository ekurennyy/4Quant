---

title: 'People Video Analytics'

---

# People Video Analytics

<div class="date">July 24, 2015</div>

## Anonymous Crowd Monitoring

Monitoring the movement of crowds is a challenging problem and important for making municipal decisions, gathing sales information, and even preventing tragic accidents which can occur when too many people attempt to go in the same direction. Standard approaches such manual counting are too slow and time-consuming, while technical approaches such as cell phone traffic monitoring too invasive.

## Drone and Stationary Cameras, Real-time In-Memory Analysis

Drone use for remote monitoring, police activities, and other services has increased rapidly in the past years. Many of these drones are equipped with video cameras producing a constant stream of information. By pairing this streaming output with our Spark Image Layer tools, live feeds from a number of sources can be processed in real-time. Furthermore since the images are being processed in real-time, it is not necessary to save any of the identifying image information alleviated many privacy concerns. Here is an example showing the activity of a street during a running event.

<div class="centered-image">
  <img src="images/drone-people-counting/dpc-001.gif" style="width: 180px;">
  <img src="images/drone-people-counting/dpc-002.gif" style="width: 180px;">
  <img src="images/drone-people-counting/dpc-003.gif" style="width: 360px;">
</div>
<br>
<div class="half-width-image">
  <img src="images/drone-people-counting/dpc-004.gif">
  <img src="images/drone-people-counting/dpc-005.gif">
  <img src="images/drone-people-counting/dpc-006.gif">
</div>

### How?

The first question is how the data can be processed. The basic work is done by a simple workflow on top of our Spark Image Layer. This abstracts away the complexities of cloud computing and distributed analysis. You focus only on the core task of image processing.

<div class="centered-image"><img src="images/drone-people-counting/dpc-007.svg" style="width: 40%;"></div>

Beyond a single camera, our system scales linearly to multiple cameras and can distribute this computation across many computers to keep the computation real-time.

<div class="centered-image"><img src="images/drone-people-counting/dpc-008.svg"></div>

With cloud-integration and *Big Data*-based frameworks, even handling an entire city network with 100s of drones and cameras running continuously is an easy task without worrying about networks, topology, or fault-tolerance. Below is an example for 30 drones where the tasks are seamlessly, evenly divided among 50 different computers.

<div class="centered-image"><img src="images/drone-people-counting/dpc-009.svg"></div>

### What?

The images which are collected by the drones at rate of 30 frames per second contain substantial dynamic information on not only persons, but vehicles, buildings, and landscapes. The first basic task is the segmentation of the people which can provide information on their number, movement, and behavior

<div class="centered-image"><img src="images/drone-people-counting/dpc-010.png"></div>

The segmented image above can be transformed into quantitative metrics at each time point. These metrics can then be processed to extract relevant quality assessment information for the tracks.

<div class="centered-image"><img src="images/drone-people-counting/dpc-011.png"></div>

The data can then be broken down into small scenes where the number and flow of people in each grouping can be evaluated.

<div class="centered-image"><img src="images/drone-people-counting/dpc-012.png"></div>

Dynamic information can be obtained by tracking the person objects (below) from one scene to the next. As seen in the image below, these are simply outlines and completely un-identifiable back to the original person. This alleviates many privacy and personal information concerns.

<div class="centered-image">
  <img src="images/drone-people-counting/dpc-013.png">
  <img src="images/drone-people-counting/dpc-014.png">
</div>

The data can also be smoothed to show more clearly trends and person counts on a single image.

<div class="centered-image"><img src="images/drone-people-counting/dpc-015.png"></div>

## Technical Aspects

### Streaming the Data

Once the cluster has been comissioned and you have the *StreamingSparkContext* called `ssc` (automatically provided in [Databricks Cloud](https://databricks.com/product/databricks-cloud) or [Zeppelin](http://zeppelin.incubator.apache.org/)), the data can be loaded using the Spark Image Layer. Since we are using real-time analysis, we acquire the images from a streaming source

<span class="code">
  val droneCam1 = TrainCameraReceiver("https://drone-8092") <br>
  val droneCam2 = TrainCameraReceiver("https://drone-8093") <br>
  val metaImageStream = ssc.receiverStream(trainCam1 ++ trainCam2)
</span>

Although we execute the command on one machine, the analysis will be distributed over the entire set of cluster resources available to `ssc`. To further process the images, we can take advantage of the rich set of functionality built into Spark Image Layer

<span class="code">
  def identifyPeople(time: Double, pos: GeoPos, inImage: Img[Byte]) = { <br>
    // Run the image processing steps on all images <br>
    val peopleOutlines = inImage. <br>
      window(3s). <br>
      run("Median...","radius=3"). // filter out the noise <br>
      run("Rolling Background Subtraction..."). // remove static objects <br>
      run("Threshold","OTSU") // threshold bright structures <br>
    val peopleShape = peopleOutlines. <br>
      morphology(CLOSE,5) // connect nearby objects <br>
      componentLabel(). // identify the components  <br>
      filter(_.area>50). // take only the larger sized <br>
      shapeAnalysis() // analyze the position and shape <br>
    // return smoothness and separation based on the segmented image <br>
    PersonInformation(size=peopleShape.area) <br>
  } <br>
  // apply the operation to all images as they come in <br>
  val peopleStream = metaImageStream.map(identifyPeople)
</span>

The entire pipeline can then be started to run in real-time on all the new images as they stream in. If the tasks become more computationally intensive, then the computing power can be scaled up and down elastically.

## Learn More

To find out more about the technical aspects of our solution, check out our presentation at the [Spark Summit](http://4quant.com/spark-east-2015) or watch the [video](https://www.youtube.com/watch?v=ohR_y7HZaHA&index=10&list=PL-x35fyliRwiy50Ud2ltPx8_yA4H34ppJ).
Check out our other demos to see how 4Quant can help you:

* [Check train tracks in real time](http://4quant.com/Railway-Check)
* [Track criminals in cars using traffic cameras](http://4quant.com/Pursuing-Criminals/)
* [Find colon cancer with capsule endoscopy](http://4quant.com/Capsule-Endoscopy/)
* [Counting Cars in Satellite Images](http://4quant.com/countingcarsdemo)
* [Finding buildings and forests in Satellite Images](http://4quant.com/geospatialdemo/)

<!-- Strange text here -->

<!-- ## Acknowledgements

<div class="news">
  <p>Analysis powered by Spark Image Layer from 4Quant, Visualizations, Document Generation, and Maps provided by:</p>
  <p>To cite ggplot2 in publications, please use:</p>
  <p>H. Wickham. ggplot2: elegant graphics for data analysis. Springer New York, 2009.</p>
  <p>A BibTeX entry for LaTeX users is</p>
  <p>
    @Book{, author = {Hadley Wickham}, title = {ggplot2: elegant graphics for data analysis}, publisher = {Springer New York}, year = {2009}, isbn = {978-0-387-98140-6}, url = {
    <a href="http://had.co.nz/ggplot2/book">http://had.co.nz/ggplot2/book</a>
  }, }
  </p>
  <p>To cite package ‘leaflet’ in publications use:</p>
  <p>
    Joe Cheng and Yihui Xie (2014). leaflet: Create Interactive Web Maps with the JavaScript LeafLet Library. R package version 0.0.11.
    <a href="https://github.com/rstudio/leaflet">https://github.com/rstudio/leaflet</a>
  </p>
  <p>A BibTeX entry for LaTeX users is</p>
  <p>
    @Manual{, title = {leaflet: Create Interactive Web Maps with the JavaScript LeafLet Library}, author = {Joe Cheng and Yihui Xie}, year = {2014}, note = {R package version 0.0.11}, url = {
    <a href="https://github.com/rstudio/leaflet">https://github.com/rstudio/leaflet</a>
  }, }
  </p>
  <p>To cite plyr in publications use:</p>
  <p>
    Hadley Wickham (2011). The Split-Apply-Combine Strategy for Data Analysis. Journal of Statistical Software, 40(1), 1-29. URL
    <a href="http://www.jstatsoft.org/v40/i01/">http://www.jstatsoft.org/v40/i01/</a>
    .
  </p>
  <p>A BibTeX entry for LaTeX users is</p>
  <p>
    @Article{, title = {The Split-Apply-Combine Strategy for Data Analysis}, author = {Hadley Wickham}, journal = {Journal of Statistical Software}, year = {2011}, volume = {40}, number = {1}, pages = {1–29}, url = {
    <a href="http://www.jstatsoft.org/v40/i01/">http://www.jstatsoft.org/v40/i01/</a>}
    , }
  </p>
  <p>To cite the ‘knitr’ package in publications use:</p>
  <p>Yihui Xie (2015). knitr: A General-Purpose Package for Dynamic Report Generation in R. R package version 1.10.</p>
  <p>Yihui Xie (2013) Dynamic Documents with R and knitr. Chapman and Hall/CRC. ISBN 978-1482203530</p>
  <p>Yihui Xie (2014) knitr: A Comprehensive Tool for Reproducible Research in R. In Victoria Stodden, Friedrich Leisch and Roger D. Peng, editors, Implementing Reproducible Computational Research. Chapman and Hall/CRC. ISBN 978-1466561595</p>
  <p>To cite package ‘rmarkdown’ in publications use:</p>
  <p>
    JJ Allaire, Joe Cheng, Yihui Xie, Jonathan McPherson, Winston Chang, Jeff Allen, Hadley Wickham and Rob Hyndman (2015). rmarkdown: Dynamic Documents for R. R package version 0.7.
    <a href="http://CRAN.R-project.org/package=rmarkdown">http://CRAN.R-project.org/package=rmarkdown</a>
  </p>
  <p>A BibTeX entry for LaTeX users is</p>
  <p>
    @Manual{, title = {rmarkdown: Dynamic Documents for R}, author = {JJ Allaire and Joe Cheng and Yihui Xie and Jonathan McPherson and Winston Chang and Jeff Allen and Hadley Wickham and Rob Hyndman}, year = {2015}, note = {R package version 0.7}, url = {
    <a href="http://CRAN.R-project.org/package=rmarkdown">http://CRAN.R-project.org/package=rmarkdown</a>
    }, }
  </p>
  <p>To cite package ‘DiagrammeR’ in publications use:</p>
  <p>Knut Sveidqvist, Mike Bostock, Chris Pettitt, Mike Daines, Andrei Kashcha and Richard Iannone (2015). DiagrammeR: Create Graph Diagrams and Flowcharts Using R. R package version 0.7.</p>
  <p>A BibTeX entry for LaTeX users is</p>
  <p>@Manual{, title = {DiagrammeR: Create Graph Diagrams and Flowcharts Using R}, author = {Knut Sveidqvist and Mike Bostock and Chris Pettitt and Mike Daines and Andrei Kashcha and Richard Iannone}, year = {2015}, note = {R package version 0.7}, }</p>
</div>
 -->
