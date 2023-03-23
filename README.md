# RAS-Mocap
A motion capture addon for blender to animate human Gait to a 3d model both in realtime and non realtime.

Motion capture, often known as mo-cap or mocap, is the method of capturing the movement of people or things. It is utilized for robot and computer vision validation, as well as in the military, entertainment, sports, and medical applications.  It refers to the process of filming human actors' motions and using that footage to create digital character models in 2D or 3D computer animation. (mocap) technology is gaining popularity every day, and there are an increasing number of potential uses for it. However, such systems are too expensive and unaffordable for private use. The Motions of human beings are complex, these motions cannot be duplicated easily.

So one of the key factors that is applied in here is Gait Anaysis of human being. Such human gait are also recorded by by enourmous machinerie, thus to create a better gait detection. 

The growh of AI&ML has been spread with many application such application is neede here to get a low cost and feasible solution to each and every 3d creators. Thus Using pose detection to create dataset from one of our Gait analysis application https://github.com/Darrshan-Sankar/Basic_Gait_analysis and collected human data with inverse kinematics of blender in order to get a more realistic motion motion from a single webcam. and euler rotation application from BlendAR mocap to copy those transforms in skeletons

# Installation
Download the file and the zipfile seperately
Install in blender by
Edit> Preferences> Addons> Install > Find the "ras.py" and Install
Same goes for the zipfile
for module installation 
Please check with requirements.txt

# POSE ESTIMATION 
For pose estimation, we utilize our proven two-step detector-tracker ML pipeline. Using a detector, this pipeline first locates the pose region-of-interest (ROI) within the frame. The tracker subsequently predicts all 33 pose keypoints from this ROI. Note that for video use cases, the detector is run only on the first frame. For subsequent frames we derive the ROI from the previous frame’s pose keypoints as discussed below.

 
Figure no:1 Human pose estimation pipeline overview.

Pose Detection by Extending BlazeFace
For real-time performance of the full ML pipeline consisting of pose detection and tracking models, each component must be very fast, using only a few milliseconds per frame. To accomplish this, we observe that the strongest signal to the neural network about the position of the torso is the person's face (due to its high-contrast features and comparably small variations in appearance). Therefore, we achieve a fast and lightweight pose detector by making the strong (yet for many mobile and web applications valid) assumption that the head should be visible for our single-person use case.
Consequently, we trained a face detector, inspired by our sub-millisecond BlazeFace model, as a proxy for a pose detector. Note, this model only detects the location of a person within the frame and can not be used to identify individuals. In contrast to the Face Mesh and MediaPipe Hand tracking pipelines, where we derive the ROI from predicted keypoints, for the human pose tracking we explicitly predict two additional virtual keypoints that firmly describe the human body center, rotation and scale as a circle. Inspired by Leonardo’s Vitruvian man, we predict the midpoint of a person's hips, the radius of a circle circumscribing the whole person, and the incline angle of the line connecting the shoulder and hip midpoints. This results in consistent tracking even for very complicated cases, like specific yoga asanas. The figure below illustrates the approach.
 
Figure no:2 Vitruvian man aligned via two virtual keypoints predicted by our BlazePose detector in addition to the face bounding box
Tracking Model
The pose estimation component of the pipeline predicts the location of all 33 person keypoints with three degrees of freedom each (x, y location and visibility) plus the two virtual alignment keypoints described above. Unlike current approaches that employ compute-intensive heatmap prediction, our model uses a regression approach that is supervised by a combined heat map/offset prediction of all keypoints, as shown below.
 
Figure no:3 Tracking network architecture: regression with heatmap supervision
Specifically, during training we first employ a heatmap and offset loss to train the center and left tower of the network. We then remove the heatmap output and train the regression encoder (right tower), thus, effectively using the heatmap to supervise a lightweight embedding.
The table below shows an ablation study of the model quality resulting from different training strategies. As an evaluation metric, we use the Percent of Correct Points with 20% tolerance (PCK@0.2) (where we assume the point to be detected correctly if the 2D Euclidean error is smaller than 20% of the corresponding person’s torso size). To obtain a human baseline, we asked annotators to annotate several samples redundantly and obtained an average PCK@0.2 of 97.2. The training and validation have been done on a geo-diverse dataset of various poses, sampled uniformly.
To cover a wide range of customer hardware, we present two pose tracking models: lite and full, which are differentiated in the balance of speed versus quality. For performance evaluation on CPU, we use XNNPACK; for mobile GPUs, we use the TFLite GPU backend.





