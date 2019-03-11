# **Finding Lane Lines on the Road** 

This project was aimed for detecting Simple Lane Lines of a road using Computer Vision techniques.

**Finding Lane Lines on the Road**
---
The Goals of the project includes:-
- Make a pipeline that finds lane lines on the road
- Reflect on your work in a written report

[test_image]: ./test_images/solidWhiteCurve.jpg "Test Image"
[gray_image]: ./writeup_images/grayImage.jpg "Gray Image"
[gaussian_image]: ./writeup_images/gaussianImage.jpg "Gaussian Image"
[canny_image]: ./writeup_images/cannyImage.jpg "Canny Image"
[masked_image]: ./writeup_images/maskedImage.jpg "Masked Image"
[hough_image]: ./writeup_images/houghImage.jpg "Hough Image"
[resultant_hough_image]: ./writeup_images/resultantHoughImage.jpg "Resultant Hough Image"
[weighted_image]: ./writeup_images/weightedImage.jpg "Weighted Image"
[output_image]: ./writeup_images/output.png "Output Image"


### Lane Detection Pipeline
---
The pipeline for detecting lane lines on the road consists of a number of steps.

**`findLines(image)`**: This is the name of my pipeline function which includes following lines and steps:-
- `test_image = np.copy(image)`: First of all I made a copy of the original image to apply all the transformations on the copy rather than the original image.

	![alt text][test_image]

- `gray_image = grayscale(test_image)`: This method included a grayscaling helper function which was used to obtain a grayscaled image for better computational complexity, i.e. work only on image intensities rather than working on the RGB colors.

	![alt text][gray_image]

- `gaussian_image = gaussian_blur(gray_image, kernel_size = 5)`: Next we do Gaussian smoothening or Gaussian blur which removes noise from an image as a measure to pre process the image before applying the Canny Edge detection algorithm as the edges are prone to noises and hence potentially wrong edges may get detected by Canny algorithm. Here I have taken a standard kernel_size of 5x5 for applying Gaussian smoothening.

	![alt text][gaussian_image]

- `canny_image = canny(gaussian_image, low_threshold = 50, high_threshold = 150)`: After that we apply one of the most important algorithm for this pipeline, the Canny Edge detection algorithm which basically finds the change in intensity of pixels, a higher change in intensity means an edge, and hence it is used to detect the edges in an image. Here I've given a low threshold of a 50 while a high threshold of 150 which denotes the minimum and maximum intensity gradient.

	![alt text][canny_image]

- `masked_image = region_of_interest(canny_image, vertices)`: After that we mask the whole image as we are only interested in the lanes and roads and not the surroundings. So we define generic vertices such that it properly encloses our area of interest for all of the images. 

	![alt text][masked_image]

- `hough_image = hough_lines(masked_image, rho = 2, theta = np.pi/180, threshold = 40, min_line_len = 100, max_line_gap = 170)`: Now we come for the real deal, we calculate the hough lines for the edges we have detected using Canny edge algorithm. Hough transformation, first converts the equation into parametric space and then uses parameters like `rho`: which is the distance of line from the origin , `theta`: the angle made by the line joining origin to the line in question with the horizontal axis in radians, `threshold`: the minimum number of intersections to detect a line, 
`min_line_len`: The minimum length of line that needs to be detected and the `min_line_gap`: The maximum gap between two points to be considered in the same line.

	![alt text][hough_image]

Inside the hough_image we've called a `draw_lines(line_img, lines)` function, which I used to extrapolate/average the patches of hough lines obtained from hough transformations.

`draw_lines` method:
- First of all iterate through each lines and each point of that line.
- Calculate slope and yIntercepts of it.
- if there is a vertical line then don't do anything, i.e. when x1=x2 just go through the next iteration.
- Remove all lines whose slope is greater than 45º and less than 30º.
- Segregate the slopes and the yIntercepts into left lane and right lane. Slopes value greater than 0 indicates right lane line while less than 0 indicates left lane lines, according to the coordinate system.
- Next if the array of slope and intercepts are not empty, calculate the mean slope and intercepts using the points in slope and intercepts array for each of the left and right lanes.
- Find the x coordinate using the boundary values of y (used in masking polygon of image), the mean slope and the mean intercept for each lane lines and draw it over the image.
- In case the array of slope or intercepts are empty for any frame draw the line same as the previous line for any of the two lane lines respectively.

	Thus, we obtain a single extrapolated line for the image using this technique along with the Hough Transformation.

	![alt text][resultant_hough_image]

- `weighted_image = weighted_img(hough_image, image)`: Finally we superimpose our hough line into the original image to get the better picture of the lane lines detetcted.

	![alt text][weighted_image]

The output lane lines detected by the pipeline for all the test images are as follows:-

![alt text][output_image]

### Identify potential shortcomings with your current pipeline
---

While doing this project, I discovered a number of shortcomings with my current algorithm which includes:-

- When the lanes are curving fast in the video, this algorithm will not be fast and accurate enough to detect the curve of the road. Although, it was able to detect minor curves of the road in this algorithm, we can see the fluctuations here too.

- When probably the lane lines are not continous enough (the lines may have faded off), the algorithm probably will find it difficult to find the line, though we can design a heuristic to see off this challenge.

- When there's a hilly terrain or an area where the roads are steep, it'll probably lose it's point of reference and may detect inaccurate paths.

- Again when there are some patches on the road like tyre marks or when we move through a forest when there is a sudden color change of the road due to irregular sunlight, the algorithm may detect inaccurate lines.

### Suggest possible improvements to your pipeline
---
I've though about some possible improvements with my approach which includes:

-  A heuristic to check for the delta change in mean slope between previous and the current frame, if the change is above some threshold value, we may take the mean of the two slopes and plot the lines out, this will in turn reduce fluctuations.

- A heuristic to give weightage to the slopes of line segments with greater length while calculating mean slope, this will ensure that the larger line segments are more significant in deciding the actual lane lines, hence increasing accuracy.
