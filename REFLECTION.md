#**Finding Lane Lines**
#### By _[**Satchel Grant**](https://github.com/grantsrb)_

The goal of this project was to make a pipeline that finds lane lines on the road and draws their projections onto the image using a Jupyter notebook environment.

---

### Reflection

####1. Pipeline Description

This pipeline broadly involves 5 steps. First, the image is converted to grayscale and slightly blurred using the GaussianBlur function from openCV. The conversion to grayscale is useful in the detection of edge gradients and is necessary for the Hough Transform later in the pipeline. The GaussianBlur smooths out noise which helps with edge detection. Ultimately, this step prepares the image for openCV's Canny edge detection function.

![Gray Scale](./examples/gray_copy.jpg "Grayscale")
_Grayscale_
![Gray Scale with Gaussian Blur](./examples/gray_blurred.jpg "Grayscale Blurred")
_Grayscale with Gaussian Blur_

Using the blurred, gray image, the pipeline uses openCV's Canny edge detection to find pixel changes above and between a custom threshold. Pixel gradients between the low and high threshold values are recorded as weak edges. Pixel gradients above the high threshold are recorded as strong edges. These edges are used later to define the location of the lane lines. This pipeline's thresholds were set through trial and error.

![Canny Edges](./examples/canny_edges.jpg "Canny Edges")
_Canny Edges_

The pipeline partitions the image to remove a portion of the excess edges. Canny edge detection detects edges other than those of the lane lines. In order to focus on the lane edges, the pipeline seperates the edges within the approximate location of each the left lane and the right lane. The pipeline does this by creating two mirrored trapezoids on the image using openCV's fillPoly function. One is for the left lane line, the other is for the right. These trapezoids are used in conjunction with numpy broadcasting to select edges from each lane without selecting edges from the rest of the image. The trapezoids extend from near the center of the image down to the hood of the car.

![Left Partition](./examples/poly_left.jpg "Left Partition")
_Left Partition_
![Right Partition](./examples/poly_right.jpg "Right Partition")
_Right Partition_

Using the partitioned edges, the pipeline uses openCV's HoughLinesP function to run a Hough transform. The transform returns lines that are common to a group of linearly placed pixels by finding graph crossings in Hough space (or parameter space). Each of the parameters are used to tune the precision of the line matching. In this pipeline, the parameters were set so that lines would connect at least 30 edge pixels, have 2 or fewer pixels of seperation in the edge, and be at least 5 pixels long on the image. These parameters were chosen under considering the balance between noise and signal. The pipeline attempts to remove random lines created from noise/obstacles while retaining lane line edge detection. The pixel and angle resolution of the transform were determined by trial and error.

![Hough Lines](./examples/hough_lines.jpg "Hough Lines")
_Hough Lines_

The final step of the pipeline is the creation of a line to sit on top of the lane in the image. For each lane, the pipeline creates a directional vector from the weighted average direction of each Hough line with a slope above a set threshold. The size of the directional contribution from each line was intentionally unnormalized. This followed from the theory that lane lines will often have large solid lines from the Hough transform which should contribute more to the average direction. In addition to the average direction, the pipeline averages the average x and y coordinates from each line into a single point. The final lane line is constructed by extending the directional vector from the averaged point to the top and bottom of the individual lane's trapezoidal space.

![Solid Lines](./examples/solid_lines.jpg "Solid Lines")
_Final Image_


###2. Shortcomings and Solutions

One large shortcoming of this pipeline is in its evaluation of shadows. This is apparent in the extra.mp4 video. Shadows are detected in the Canny edge detection and thus can be detected in the Hough transform which can affect the average direction of the constructed lane line. This could likely be improved by giving extra weight to Hough lines that overlay pixels of white and yellow color. Another potential improvement could be to include the previous frame's lane line in the average directional vector. This second approach could additionally help stabilize the lane line as discussed later.

Another shortcoming is the detection of random objects near the lane lines. Although it may be helpful for an autonomous vehicle to detect random objects in the road, it is best to keep detection of lanes seperate from detection of random objects. Excluding objects in lane detection is a similar problem to that of excluding shadow detection. It can likely be solved in the same ways.

One final shortcoming is the instability of the drawn lane line. The lines are shakey which is largely due to the new series of calculations at every frame of the video. The most effective solution would likely be to include the previous frame's drawn lane line in the average directional vector.

