# Object-Detection-and-Motion-Tracking

Table of Contents
Video Data in MATLAB............................................................................................................................................. 1
Label Ground Truth Data.......................................................................................................................................... 6
Train a Detector........................................................................................................................................................ 7
Detect Objects with the Detector.............................................................................................................................. 8
Visualizing Detections...............................................................................................................................................9
Improving an Object Detector.................................................................................................................................10
Count Detected Objects..........................................................................................................................................14
Detect Objects in Each Frame of a Video...............................................................................................................15
Track an Object.......................................................................................................................................................17
Video Data in MATLAB
Create a video reader that you can use to access video data, including:
• individual frames, and
• metadata about the video.
v = VideoReader("turtles.avi")
v = 
 VideoReader with properties:
 General Properties:
 Name: 'turtles.avi'
 Path: 'C:\Users\gkennedy\OneDrive - MathWorks\Documents\CV OT\2023Oct update\orcv_course_files'
 Duration: 3.3333
 CurrentTime: 0
 NumFrames: 100
 Video Properties:
 Width: 400
 Height: 600
 FrameRate: 30
 BitsPerPixel: 24
 VideoFormat: 'RGB24'
Access properties of a video reader with dot notation. 
v.NumFrames
ans = 100
v.CurrentTime % currrent time is 0 because no frames have been read
ans = 0
Read the next frame of a video with readFrame. The video reader keeps track of where you are in the video.
fr = readFrame(v);
v.CurrentTime % current time advances because the first frame has been read
ans = 0.0333
1
imshow(fr)
fr = readFrame(v);
v.CurrentTime % current time advances again because the second frame has been 
read
ans = 0.0667
imshow(fr)
2
Read the 50th frame of a video. 
fr50 = read(v,50);
v.CurrentTime % current time is about half way through the video
ans = 1.6667
imshow(fr50)
3
Read the last frame of a video. 
fr = read(v,Inf);
v.CurrentTime % current time is the same as the duration because we are at 
the end of the video
ans = 3.3333
v.Duration
ans = 3.3333
4
At the end of the video, when there are no more frames left to read, trying to read the next frame produces an 
error. 
% fr = readFrame(v); 
Set the CurrentTime property to 0 to read frames from the beginning. 
v.CurrentTime = 0;
Since each frame is an image, video processing often comes down to image processing on each frame. You 
can use a while loop to read frames until there are none left to read.
% create video reader
while hasFrame(v)
 % code to read the next frame
 % code to process the frame
end
Use the hasFrame function to determine if there are frames left to read. This code reads and displays each 
frame.
v = VideoReader("turtles.avi")
v = 
 VideoReader with properties:
 General Properties:
 Name: 'turtles.avi'
 Path: 'C:\Users\gkennedy\OneDrive - MathWorks\Documents\CV OT\2023Oct update\orcv_course_files'
 Duration: 3.3333
 CurrentTime: 0
 NumFrames: 100
 Video Properties:
 Width: 400
 Height: 600
 FrameRate: 30
 BitsPerPixel: 24
 VideoFormat: 'RGB24'
while hasFrame(v)
 % read the next frame
 fr = readFrame(v);
 % image processing, here convert to grayscale
 fr = im2gray(fr);
 
 % display the video as an animation
 imshow(fr)
 drawnow
end
5
Label Ground Truth Data
Training an object detector requires ground truth data as a reference. Ground truth data consists of:
1. Images
2. Labels for objects in the images
3. Locations for objects in the images
You can label ground truth data with the Video Labeler App, which you can open in the Apps tab under Image 
Processing and Computer Vision. Or, you can run the videoLabler command.
6
% videoLabeler
This code loads labeling data exported from the Video Labeler App.
load lotsOturtles_groundTruth
gTruth
gTruth = 
 groundTruth with properties:
 DataSource: [1×1 groundTruthDataSource]
 LabelDefinitions: [1×5 table]
 LabelData: [43×1 timetable]
gTruth.DataSource
ans = 
groundTruthDataSource for a video file with properties
 Source: ...rcv_course_files\lotsOturtles_forVideoLabeling.avi
 TimeStamps: [43×1 duration]
The output of the Video Labeler is a ground truth object with labels and locations for each object you labeled. 
Training a detector requires image files to be extracted from the video.
if ~isdir("trainingImages")
 mkdir("trainingImages")
end
If you rerun this code with a different SamplingFactor value, you need to delete the images in the 
trainingImages folder.
[imList,boxLabels] = objectDetectorTrainingData(gTruth, ...
 SamplingFactor=4, ...
 NamePrefix="turtleFrame", ...
 WriteLocation="trainingImages");
Write images extracted for training to folder: 
 trainingImages
Writing 9 images extracted from lotsOturtles_forVideoLabeling.avi...Completed.
The last step in creating ground truth data is to combine the labels and the list of image files into a single 
datastore.
imsWithBoxLabels = combine(imList,boxLabels);
Train a Detector
Training over more stages may improve the detector's accuracy. Here, train the detector in five stages. 
detector = trainACFObjectDetector(imsWithBoxLabels,NumStages=5)
ACF Object Detector Training
The training will take 5 stages. The model size is 13x23.
7
Sample positive examples(~100% Completed)
Compute approximation coefficients...Completed.
Compute aggregated channel features...Completed.
--------------------------------------------
Stage 1:
Sample negative examples(~100% Completed)
Compute aggregated channel features...Completed.
Train classifier with 21 positive examples and 105 negative examples...Completed.
The trained classifier has 19 weak learners.
--------------------------------------------
Stage 2:
Sample negative examples(~100% Completed)
Found 105 new negative examples for training.
Compute aggregated channel features...Completed.
Train classifier with 21 positive examples and 105 negative examples...Completed.
The trained classifier has 19 weak learners.
--------------------------------------------
Stage 3:
Sample negative examples(~100% Completed)
Found 68 new negative examples for training.
Compute aggregated channel features...Completed.
Train classifier with 21 positive examples and 105 negative examples...Completed.
The trained classifier has 34 weak learners.
--------------------------------------------
Stage 4:
Sample negative examples(~100% Completed)
Found 1 new negative examples for training.
Compute aggregated channel features...Completed.
Train classifier with 21 positive examples and 105 negative examples...Completed.
The trained classifier has 34 weak learners.
--------------------------------------------
Stage 5:
Sample negative examples(~100% Completed)
Found 1 new negative examples for training.
Compute aggregated channel features...Completed.
Train classifier with 21 positive examples and 105 negative examples...Completed.
The trained classifier has 34 weak learners.
--------------------------------------------
ACF object detector training is completed. Elapsed time is 1.7249 seconds.
detector = 
 acfObjectDetector with properties:
 ModelName: 'turtle'
 ObjectTrainingSize: [13 23]
 NumWeakLearners: 34
Detect Objects with the Detector
load anotherFrame
[bbox,score] = detect(detector,laterFrame)
bbox = 7×4
 197 474 23 13
 53 49 30 17
 63 226 30 17
 40 47 32 18
 46 53 32 18
 74 222 42 24
 297 67 46 26
score = 7×1
 51.6772
 74.6286
8
 13.4356
 24.0355
 66.2742
 79.3308
 93.6366
Visualizing Detections
img = insertObjectAnnotation(laterFrame,"rectangle",bbox,score);
imshow(img)
9
Zoom in on objects detected more than once.
xlim([0 200])
ylim([0 300])
Improving an Object Detector
There are three things you can do to improve an object detector:
1. Improve the training data: More is better. Make sure the ground truth images are representative of the 
environment where you will detect the objects.
10
2. Tweak the training parameters: For the ACF object detector, you increased the number of training stages 
from four to five.
3. Post-process detections: Remove incorrect or overlapping detections so that each object is detected 
exactly once.
Two ways to post-process detections are
• thresholding based on the detection scores
• selecting the strongest detection for a single object
Visualize the histogram of scores to see if you can determine a cut-off to threshold the detections.
histogram(score,10)
11
bbox = bbox(score>30,:);
score = score(score>30);
img = insertObjectAnnotation(laterFrame,"rectangle",bbox,score);
imshow(img)
xlim([0 200])
ylim([0 300])
12
Thresholding removes weaker detections. When an object is detected twice, you can remove detections based 
on overlap and/or defining a maximum number of objects.
[selectedBbox,selectedScore] = selectStrongestBbox(bbox,score, ...
 OverlapThreshold=0.1, ...
 NumStrongest=4);
img2 = insertObjectAnnotation(laterFrame,"rectangle", ...
 selectedBbox,selectedScore);
imshow(img2)
13
Count Detected Objects
numBoxes = size(selectedBbox,1)
numBoxes = 4
str = numBoxes + " turtle(s) detected"
str = 
"4 turtle(s) detected"
imgCounted = insertText(img,[250 550],str);
imshow(imgCounted)
14
Detect Objects in Each Frame of a Video
% create video reader
while hasFrame(v)
 % code to read the next frame
 % code to process the frame
end
v = VideoReader("lotsOturtles_firstSec.avi");
while hasFrame(v)
 % read the next frame
 frame = readFrame(v);
15
 % image processing, here detect and count objects
 [bbox,score] = detect(detector,frame);
 bbox = bbox(score>30,:);
 score = score(score>30);
 [selectedBbox,selectedScore] = selectStrongestBbox(bbox,score, ...
 OverlapThreshold=0.1);
 numBoxes = size(selectedBbox,1);
 str = numBoxes + " turtle(s) detected";
 img = insertObjectAnnotation(frame,"rectangle", ...
 selectedBbox,"Turtle: " + selectedScore);
 img = insertText(img,[250 550],str,TextColor=[1 1 0]);
 % display the video as an animation
 imshow(img)
 drawnow
end
16
Track an Object
The tracking section uses a clip with an occlusion. The detector has already been trained and the initial 
detection calculated.
load ACFdetector4tracking
v = VideoReader("turtle.avi");
frame = readFrame(v);
[bbox,score] = detect(detector,frame);
bbox = bbox(score>95,:);
17
score = score(score>95);
bbox = selectStrongestBbox(bbox,score, ...
 NumStrongest=1);
A tracker uses a single point to track an object, so first calculate the centroid of the detection.
centroid = [bbox(1)+bbox(3)/2 bbox(2)+bbox(4)/2];
frame = insertShape(frame,"filled-circle",[centroid 50],Color="green");
imshow(frame) 
Now, you need to define assumptions about the motion of your object and initialize a Kalman filter.
motionModel = "ConstantVelocity";
initialLoc = centroid;
initialError = [1 25];
motionNoise = [1 25];
measureNoise = 100;
kf = 
configureKalmanFilter(motionModel,initialLoc,initialError,motionNoise,measureNoise);
On each frame, you use the Kalman filter to follow a predict-detect-correct workflow to track an object.
% predict
trackedLoc = predict(kf)
% read the next frame and detect the centroid
% this code often includes post-post processing steps between detection and centroid calculation
frame = readFrame(v);
18
[bbox,score] = detect(detector,frame);
% ...
centroid = [strongestBbox(1)+strongestBbox(3)/2 strongestBbox(2)+strongestBbox(4)/2];
% correct
trackedLoc = correct(kf,centroid)
frame = insertShape(frame,"filled-circle",[trackedLoc 20],Color="red");
imshow(frame)
Use a while loop to track an object through a video.
% create video reader
while hasFrame(v)
 % predict the location in a frame using the Kalman filter
 % read the next frame
 % detect object centroid
 % correct and update the tracker/prediction
end
v = VideoReader("turtle.avi");
while hasFrame(v)
 trackedLoc = predict(kf);
 frame = readFrame(v);
 [bbox,score] = detect(detector,frame);
 bbox = bbox(score>95,:);
 score = score(score>95);
 if ~isempty(bbox)
 strongestBbox = selectStrongestBbox(bbox,score,NumStrongest=1);
 centroid = [strongestBbox(1)+strongestBbox(3)/2 
strongestBbox(2)+strongestBbox(4)/2];
 frame = insertShape(frame,"filled-circle",[centroid 50],Color="green");
 trackedLoc = correct(kf,centroid);
 end
 frame = insertShape(frame,"filled-circle",[trackedLoc 20],Color="red");
 imshow(frame)
 drawnow
end 
19
20
