# MSD Analysis  

## 1. A Brief Description 

<div style="border-left: 7px solid #ffe95c; background-color: #fff5b8; padding: 10px;">
    This MATLAB script provides a comprehensive, all-in-one workflow for analyzing particle trajectory data. It's designed to process raw tracking data from a CSV file, guide the user through an interactive data filtering process, perform Mean Squared Displacement (MSD) analysis, and export publication-quality plots and data.
</div>
<br>

The workflow is built around the powerful *@msdanalyzer* class developed by Jean-Yves Tinevez.

### 1.1 Key Features 

- **Data Import**: 
This script expects the particle trajectory data provided in a standard CSV file.

- **Interactive Curation**: 
A graphical user interface (GUI) allows you to inspect each trajectory and its corresponding MSD curve, making it simple to discard invalid tracks.

- **Velocity Analysis**: 
Automatically calculates and visualizes instantaneous velocity distribution and velocity correlation for diagnostic purposes.

- **MSD Calculation & Fitting**: Computes individual and mean MSD curves, performs a linear fit to determine the diffusion coefficient ($D$), and assesses the goodness of fit ($R^2$).

- **Automated Export**: 
Generates clean CSV files and publication-ready plots of the final Mean MSD data with error bars.

### 1.2 Prerequisites 

- **MATLAB**: 
The scirpt was written via a recent version of MATLAB (R2025a)
- **@msdanalyzer Class**: 
This script is a wrapper for the @msdanalyzer class, which is 

## 2. A step-by-step tutorial

### 2.1  **Prepare Your Data**

You can use tracking plugins (e.g. TRACKMATE) in ImageJ, Video Spot Tracker, or Machine Learning Model to acquire the coordinates information of your Micro/Nano-motors and store it in a csv file as the following figure shows: 
 <img src="./Images/csv%20file%20storing%20all%20the%20tracjectories.png" alt="Plot" width="450">

### 2.2 Data Loading and Preprocessing

#### Define the conversion factor, time interval, and file path (code section 1)
```
% --- Section 1. Define constants and specify file paths ---
% --- 小节 1. 定义常数和指定文件路径
disp('--- Section 1: Define Constants and Specify File Paths ---');

% Define the conversion factor from pixels to micrometers
% 定义换算常数（像素 -> 微米）
microns_per_pixel = 273.2/1436;

% Define the time interval between frames in seconds (e.g., 30 fps -> 1/30 s)
% 定义两帧的时间区间（30 帧/秒对应的时间区间是 1/30）
time_interval = 1/30; 

% Define the path to your data file
% 定义粒子轨迹文件路径
filePath = 'Trajectory Data\combined_trajectories_for_msd (10mM H2O2)-(1-300_1500-1849 3500-3799frames).csv';
```
**Remark**: You need to obtain the conversion factor(e.g. pixels to micrometers) of your microscope.

#### Load and read CSV file (code section 2)
Code section 2 first reads your CSV data into a MATLAB table. It then converts all pixel coordinates to micrometers and organizes the data into a cell array named tracks, where each cell contains the full trajectory (time, x, y) for a single particle.

```
% --- Section 2. Read the CSV data into a table ---
% --- 小节 2.读取粒子轨迹CSV 文件

disp('--- Section 2: Read the CSV Data into a Table ---');
try
    T = readtable(filePath); % T is a table loaded from your csv (you can check it in the workspace)
catch ME % when errors occur, MATLAB creates a MEException object, which cntains useful fields like ME.message
    error('Failed to read the file. Please check the filePath variable. Error: %s', ME.message);
end
disp('Data loaded successfully.');
```


The csv file to be processed should contain the information of each particle's ID, frame number, and x/y-coordinates. As the following example csv file shows: 

<img src="./Images/csv%20file%20storing%20all%20the%20tracjectories.png" alt="Plot" width="450">

Once the csv is loaded successfully, you can check the table storing all information in the worksapce in MATLAB: 

<img src="./Images/Table%20T.png" alt="Plot" width="450">

#### Store all trajectories in a cell array (code section 3)

**tracks** is an $N \times 1$ (N = numParticles, i.e., the number of particles) cell array which looks like this: 

<img src="./Images/Cell%20Array_tracks.png" alt="Plot" width="450">

You can check each entry by simply clicking it: 

<img src="./Images/Single%20Entry%20in%20tracks.png" alt="Plot" width="450">

The first column is the time vector, and the second and the third are the x-coordinates and y-coordinates, respectively.

#### Normalize trajectories (code section 4)
Since the starting point may vary drastically from point to point, which would affects the plotting and make visualization difficult, hence for better view, normalization shifts the starting point of all particles to the origin $(0 , 0)$. 


### 2.3 Interactive Trajectory Curation (code section 5)
Code section 5 is designed to filter invalid trajectories. A window will pop up, showing a diagnostic plot for each particle trajectory, one by one. Once the filtering is completed, the valid tracks are saved and exported as a CSV file called *valid_tracks*. 

- Use the buttons at the bottom of the window to decide the fate of each trajectory:
    - Press *Delet* to delet invalid trajectories;
    - Press *Next* to continue inspection;
    - Press *Save* to save the desired figure;

    <img src="./Images/Track_Index_1_Particle_ID_0.png" alt="Plot" width="700">

### 2.4 Visualizing Instantaneous Velocity Distribution & Velocity Correlation
Once you have finished curating your tracks, the script automatically generates two diagnostic plots based on the remaining trajectories:
- **Veocity Distribution**: A histogram of the instantaneous velocities ($v_x ~\text{and}~ v_y$). For pure Brownian motion, this distribution should be a Gaussian distribution centered at zero.

    <img src="./Images/velocity%20distribution.png" alt="Plot" width="700">

- **Velocity Correlation**: A plot showing how the velocity is correlated over time. For pure Brownian motion, this should be zero for all time except at $\Delta t=0$.

    <img src="./Images/velocity%20correlation.png" alt="Plot" width="700">

### 2.5 Visualizing MSD, MeanMSD, and fitting curve

- *plotMSD* plots all the MSD curves.

    <img src="./Images/All MSD-2.png" alt="Plot" width="700">

- *plotMeanMSD* plots the weighted mean of all MSD curves with error bars, where the gray-shaded region represents the standard deviation over all MSD curves.

    <img src="./Images/Mean MSD.png" alt="Plot" width="700">

- You can add the linear fitting or the quadratic fitting curve on the MeanMSD figure: 

    <img src="./Images/MeanMSD and Linear Fitting Curve.png" alt="Plot" width="700"> 

- The calculated diffusion coefficient and Goodness of fitting will be displayed in the command window:

    <img src="./Images/Fitting Info.png" alt="Plot" width="700"> 


### 2.6 Export Results
Finally, the script automatically generates the final outputs based on the filtered and analyzed data. For specified fit_percentage and time_limit, you will obtain:
- **A CSV file** (e.g. MSD_Fit_on_First_5_Percent_Filtered_2.0s): This file contains the time data, Mean MSD, Standard Error of the Mean (SEM), the Diffusion Coefficient (D) and the goodness of fitting ($R^2$). You can find it in a folder named *MSD Data*.

    <img src="./Images/exported result.png" alt="Plot" width="700"> 

- **An MSD figure and a fitting curve**: A polished figure showing the Mean MSD with SEM error bars, a clear title reporting the diffusion coefficient, and properly labeled axes. 

    <img src="./Images/MSD 20%25 and Quadratic Fitting.png" alt="Plot" width="700"> 

You can also check the result displayed in the command window: 

<img src="./Images/fitting & exporting info.png" alt="Plot" width="700">

## Citing the Tool: msdanalyzer

- This script relies on the @msdanalyzer class, which was developed by Jean-Yves Tinevez. For more information on @msdanalyzer, please go to [Github](https://github.com/tinevez/msdanalyzer) or [MATLAB community](https://ww2.mathworks.cn/matlabcentral/fileexchange/40692-mean-square-displacement-analysis-of-particles-trajectories).

- I sincerely appreciate Jean-Yves’ contribution in developing the tool @msdanalyzer.

## References and Online Resources 

- [A Practical Guide to Analyzing and Reporting the Movement of Nanoscale Swimmers](https://pubs.acs.org/doi/10.1021/acsnano.1c07503?ref=PDF) - Wei Wang, and Thomas E. Mallouk*.

- [微纳米马达运动分析入门 - 哈工大(深圳) 王威](https://www.bilibili.com/video/BV1By4y1T7MW/?spm_id_from=333.337.search-card.all.click&vd_source=0bb66af6aa7294217a1c04b9157af612)

