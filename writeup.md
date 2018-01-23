# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals of this project were the following:

* Make a pipeline that finds lane lines on the road
	* The file P1.ipnb provides the Jupyter iPython notebook and source-code to find lanes.
* Reflect on your work in a written report
	* The reflection is contained in this writeup

[image1]: ./processed_images/processed_solidWhiteCurve.jpg "WhiteCurve"
[image2]: ./processed_images/processed_solidYellowCurve2.jpg "YellowCurve2"
[image3]: ./processed_images/video_still.png "Final"

---

# Reflection

## 1. Descrition of Pipeline:

The pipeline consisted of 6 stages. However, the order of execution is critical to get the desired result.

### Stages of the Pipeline
1. **GrayScale the image**

	We need to graycale first, to focus on "light" colors, close to whites.
	
2. **Do Gaussian Blur on the gray-scaled image**

   Then, gaussian blur to reduce stark differences, so regions of similar color can be obtained.
   I picked a kernel size of 5 pixels, because it seemed like a good fit for the 540x960 pixels image.
   Picking 3 or 1 would have been too small.

3. **Perform Canny Edge-Detection**

   Now we do Canny Edge-Detection. I picked 50, and 150 for the intestity-gradient thresholds, because they gave me a good result during the lectures.

4. **Mask the image with Region-of-Interest**

   Picking a region-of-interest basically involved masking the image with a trapezoid, and discarding the area outside of it. To pick the vertices, I experimented with a few multiples of the dimensions of the image, until all of the sample images were being picked up correctly. The key here was to get the furthest-ahead lane within the trapezoid, which would get missed if the trapezoid was too narrow clsoe to the top.
   
   * **Observation: Masking with the Region-of-Interest before Edge-Detection**
   
     My initial thought was to improve processing-time and do the masking before Canny Edge-Detection.
     However, this was the incorrect approach, because the boundaries of the trapezoid would get picked up as edges, as well.

5. **Generate Hough Lines and Draw those Lines**
   
   Generating Hough lines is basically an exercise in dimensional transformation, and finding straight lines i.e. from the edges, and discarding the rest of edges.
   
   However this can get tricky if the parameters are not set correctly. Similar to picking the region-of-interest, the challenging part here was to be able to select the dashed lanes which were the furthest-ahead. This required careful settings for all parameters, expecially for min_line_len of 7 pixels, and max_line_gap of 3 pixels. This allowed for the small furtheset-ahead dashed lanes to be picked-up.
   
   The following are the outputs of generating Hough-lines and drawing them on 2 of the images. These images have solid and dashed lanes on opposite sides.
   
   ![White Curve Image][image1]
   
   ![Yellow Curve2 Image][image2]

   * **Modification of draw_lines()**

     The original draw_lines() only displayed all the lines that were obtained, as in the images above. This has one very big drawback: it fails to extend the dashed-lane side to the bottom of the screen. 
     
     To overcome this drawback, I had to make the following 2 changes:
     
     + **Bounding the slopes of the lines:** By restricting the slopes of the lines, we can only select the lane markings, which are bound to point to the apex, and thus will never be horizontal or vertical. 

         Initially, I picked a very wide range for the left and right lanes, between 0.1 and 10. This worked well for the test images, but sometimes in the video the lanes seemed to jitter. I paused the video at the exact instance that happened, and noticed that sometimes the middle of the lane had a very light gray ashphalt, which would seem as a lane. 
         
         To improve it then, I tightened the slope even further, between 0.2 and 5.0 (rise / run), and that stopped the middle of the lane from being recognized as a valid lane-marking.
       
     + **Polyfit linear regression:** After obtaining my left and right lanes, now it was time to extrapolate the dashed-lanes to the bottom of the screen. 

         So I collected all the x's and y's and the highest 'y' of a line. I did a linear-regression on the x's and the y's, using polyfit, and then simply obtained the x-coordinates, for the bottom of the screen 'y', and for the highest 'y'. This gave me my full lane on the dashed-lane side. 
     
         The solid lane was already being picked up correctly, but this logic works on both solid and dashed-lanes, and so it was applied to both, and results obtained.
         
         I also incremented the thickness 20 pixels more to get a more pronounced lane identifier.
         
     
6. **Impose the weighted lines onto the original image**

   Finally, the weighed lines were added onto the image, with the default settings.

   Here is a still from the video displaying the final result.
   ![Video Still][image3]


## 2. Identify potential shortcomings with your current pipeline


* Will not work if road at incline / lane markings outside of slopes defined
* Also when road itself is gray/close to white



## 3. Suggest possible improvements to your pipeline

* Dynamic slope detection?
* More ?
