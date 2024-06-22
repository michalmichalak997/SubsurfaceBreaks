https://zenodo.org/badge/DOI/10.5281/zenodo.12375568.svg 
# Broken terrains v. 1.0

This software is intended to detect faults on homocline terrains. The software consists of three programs:

1. Generating labelled synthetic terrains based on random parameters with bounds introduced by a user (C++).
2. Detecting faults for synthetic terrains using Support Vector Machine algorithm (Python).
3. Detecting faults for real terrains with calculated attributes (C++).

We will now explain the components of the framework.

## ad. 1 Generating labelled synthetic terrains based on random parameters with bounds introduced by a user.

The first piece of software (Broken_synthetic_terrains) uses CGAL library to generate an arbitrary number of geological homoclines with calculated attributes. The homoclines are represented as triangulated terrains and the analysis is done for the faces of the triangulation. The calculated attributes can represent either local geometric attributes such as the orientation of normal and dip vectors or neighborhood analysis including distances between a specific triangle and its neighbors. The user should specify the bounds which determine ranges of intervals for the parameters. Then, random numbers from the uniform distribution are created from the determined intervals. We suggest giving ranges that best mimic the real terrains to be analyzed in the third step.

![program1_documentation](https://github.com/michalmichalak997/MLgeom/assets/28152295/4343e70e-b13a-450f-8623-30dc1d4cfe1f)

In the attached screenshot, we can see that the user requested 300 files (terrains). Then, the size of the terrains is constant because the lower bound (1) is equal to the upper bound (1). The dip angle will not be constant: it will vary between 2 and 5 degrees. Next, the dip direction will also vary between 40 and 70 degrees. The number of points in the triangulated terrains will be the same (100). The noise applied to the surface will be a fraction (1-4%) of the elevation difference of the terrain. The fault throw will be a fraction (5-10%) of the maximum elevation difference within the triangulated terrains. 

![program1_documentation1](https://github.com/michalmichalak997/MLgeom/assets/28152295/3e65ad31-5762-4810-ba8b-ead86269f08d)

Here, we can see a portion of the dataframe resulting from running the first program. Each row corresponds to a triangle of the triangulation, and the key element is the label (-1 or 1) which distinguishes between fault-related and fault-unrelated triangles.

## ad. 2 Detecting faults for synthetic data. 

A Python script (Broken_terrains_training_testing_evaluating) is used to apply Support Vector Machine to the synthetic data set. 

![program2_documentation](https://github.com/michalmichalak997/MLgeom/assets/28152295/6276cee8-caaa-4c60-8c44-8480e2d6599b)

The above screenshot presents the key step in the data preparation for supervised classification: the distances with neighbors are sorted (Angle_Max, Angle_Min, Angle_intermediate). This ensures that we eliminate randomness from the analysis, since the initial indices of the neighbors (first, second, third) correspond only to the counterclockwise order of the neighbors (https://graphics.stanford.edu/courses/cs368-04-spring/manuals/CGAL_Tutorial.pdf).

## ad. 3 Detecting faults for real terrains with calculated attributes.

To finish the fault detection pipeline, the user must calculate terrain attributes for real data. The program from the first step cannot be used because it was intended to create synthetic terrains with labels. And our objective is now to use the fine-tuned program from step 2 to predict fault-related triangles for real data based on geometric attributes (orientation of normal/dip vectors with neighborhood analysis). Please use the Broken_real_terrains code to calculate attributes for your real data. You will need to remove the first line "Index of the surface:0" from the "_output" file to proceed with the Python script.

## Software used.

CGAL library v. 4.8.
CMake (3.28.2)
Microsoft Visual Studio 2022.
Microsoft  Visual C++ 2015-2022 Redistributable (x64) - 14.38.33135.

### For running executables:

To allow portability of the programs, the C++ computer code can be compiled in Release mode to generate executables. 
The executables were run on a Windows virtual machine: Windows 10 Home, x64, Version: 22H2, OS build 19045.3803, RAM: 2048 MB.  Additionally, the following software is necessary to run the executables: Microsoft  Visual C++ 2015-2022 Redistributable (x64) - 14.38.33135. In the directory with the executable, there should be the libgmp-10.dll file available. This file is attached in this repository.

## Installation. 

The user has to install the CGAL library to run the attached computer codes specified in points 1 and 3. Newer versions of the CGAL library can be installed using the vcpkg manager (https://www.cgal.org/download/windows.html). However, since we use the 4.8 version, this option may not be available. And we do not guarantee that the computer code will be correctly executed with newer versions of CGAL (>4.8). Therefore, the best solution would be to install the CGAL (4.8) library using CMake GUI (3.28.2). We note that to use the programs, it is not mandatory to have QT installation completed.

## Tests 

To increase quality of the created programs, we performed tests with regard to the critical elements of the framework. They are described below and attached in the catalogue as well :

1. It tests whether the points are sorted according to the Z (elevation) value. It is achieved using lambda expressions. This is important in terms of calculating the random fractions of the elevation differences used to determine noise and fault throw levels.
![test_program_points](https://github.com/michalmichalak997/MLgeom/assets/28152295/0d85ab54-2150-4b5f-bbc6-d2d75195db6f)

As can be seen, the program properly sorted points according to the elevation. 

2. It tests whether the orientations supplied in the dip angle/dip direction format are properly converted into vector Cartesian coordinates by back-converting the vectors into the dip angle/dip direction format. A toleration for deviations double eps=0.01 due to floating-point round-off errors is introduced.

![test_program_converted2](https://github.com/michalmichalak997/MLgeom/assets/28152295/f052e2a3-63fd-495a-99b7-a44ad10582b0)

As can be seen, angles 3 and 363 are considered equal. This is to allow users to sample from intervals crossing N (360=0) direction. For example, the left range of the azimuth can be 355 and the right range can be 5 (which is equal to 365). 

3. It tests whether the distances calculated in C++ program are equal to those calculated in R. For Euclidean and cosine distances the differences are very small (E-06). For angular distances the errors are greater (E-04-E-02). This effect can be likely attributed to different implementations of acos() function between C++ and R. We believe that the errors are not significant and can be tolerated for geological applications such as detecting faults on geological terrains. However, if precision is critical, we suggest rewriting the code using other types, for example long double instead of double.

![image](https://github.com/michalmichalak997/MLgeom/assets/28152295/19e77aa5-965c-4052-83a5-12ea19cd6467)

Here, we can see that the differences for cosine distance between C++ and R are very small. 

![test_distances_r2](https://github.com/michalmichalak997/MLgeom/assets/28152295/0fc99891-f976-426c-a3ee-40f96f6b4ac6)

Here, we can see that the differences for angular distance are greater, probably due to different implementations of acos() function between C++ and R.

4. Calculation of distances between a triangle and its neighbors. We prepared 6 data points resulting in a triangulation 4 faces. We compared the results with those obtained in a spreadsheet. We didn't obtain any serious issues with the results. We note that the dip vector representation is not applicable for horizontal observations because the dip vector is not uniquely defined. Therefore, distances calculated for dip-vector-representations of horizontal observations are invalid. To create greater awareness of this problem, dip vectors of horizontal observations are represented as [-9999,-9999,-9999]. The code, reports and the spreadsheet are attached as separate files.

![test_4-points](https://github.com/michalmichalak997/MLgeom/assets/28152295/66db07ce-5571-417e-9447-95f6ddb398af)

A sketch of the distribution of the data points. In the central part, there is a triangle which is the only triangle having more than one finite neighbor.


![test_4](https://github.com/michalmichalak997/MLgeom/assets/28152295/dffd7160-4be2-47b2-b19f-899ad8b50509)

Here, we can see that the triangle has only one finite neighbor. This finite neighbor is a horizontal triangle. Therefore, its dip vector has the following form:  [-9999,-9999,-9999].



5. Visual inspection of collinearity
   
![collinearity](https://github.com/michalmichalak997/MLgeom/assets/28152295/8b58fb5b-8045-4e06-82b7-c39590f2f43f)

Here, we can see that the coefficient of collinearity (DOC) meets the expectations of pointing to both equilateral (DOC around 0.5) and collinear configurations (DOC close to 1.0).

