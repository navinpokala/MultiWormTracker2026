# MultiWormTracker

MATLAB-based worm tracking software, optimized for **MATLAB 2024a** (tested on **R2024b** and should work on later versions).

---

## Setup & Installation

1. **Toolboxes:** Make sure that you have the MATLAB **Image Processing** and **Statistics** toolboxes installed.
2. **Navigate to Code:** Launch MATLAB and `cd` to wherever you put the code.
   > **Important:** I **STRONGLY** recommend against using spaces in directories or filenames.
3. **Configure Path:** Edit `MultiWormTracker_setup.m`:
   * Edit `MultiWormTracker_CODE_PATH` to where you put the source code directory.
4. **Windows Only:** Copy the contents of `put_into_C_windows_system32` to `C:\Windows\System32`.
5. **Reset MATLAB Paths:** Temporarily remove any other non-core MATLAB paths using the **Set Path** menu in the toolbar (**Set to Default**, then **Close**). This is necessary to avoid conflicts between function names, etc.
6. **Initialize Session:** Run `MultiWormTracker_setup` in MATLAB each time you want to use the code:
   ```matlab
   MultiWormTracker_setup
   ```
   * This will temporarily set the program path to `MultiWormTracker_CODE_PATH`.
   * These changes are **NOT** saved; you will need to run `MultiWormTracker_setup` each time unless you manually save the path. However, you will not need to edit `MultiWormTracker_setup.m` again.

---

## Video Requirements & Compression

* **Format & Resolution:** Your movie should be an `.avi` file recorded at **least 3 Hz** with at least **512 × 512** resolution (around **300 pixels per mm²**).
* **Compression:** Try using movies compressed with `force_avi_file_to_defined_time_and_compress_mpeg` (a MATLAB wrapper for `ffmpeg`):
  ```matlab
  force_avi_file_to_defined_time_and_compress_mpeg(filename, outputfilename, duration_sec, framerate)
  ```

---

## Experimental Setup & Calibration

### Copper Rings (Default)
By default, the software looks for a **28 mm × 28 mm** frame (the copper ring):
1. Punch a 28 mm × 28 mm hole in Whatman 3MM paper ([example punch](http://www.acherryontop.com/shop/cat/punches/184898)).
2. Trim a frame.
3. Briefly soak the frame in **20 mM CuCl₂** and place on an NGM plate.
4. Add the worms.
5. Position the plate such that the outer edge of the ring is just outside the field of view (see `square_copper_ring.pdf` for an example).
   * *If it cannot find a ring, a GUI will pop up.*

### Alternative Calibration Options
* **6 mm Holepunch:** If you draw a circle somewhere using a standard **6 mm diameter holepunch**, it can use that as well (include the `'holepunch'` option).
* **Separate Scale Movie:** Instead of drawing on your plate of interest, you can take a short movie (~10 frames) of some object whose dimensions are known, and include that as the `'scale'` argument.

---

## MultiWormTracker Usage

To track, `cd` to the directory containing your movies, or the directory just above.

```matlab
MultiWormTracker(argument1, other_arguments)
```

### `argument1` Options
`argument1` can be:
* A movie filename
* A directory of movies *(automatically tries to detect the ring and worms for all movies, asks for user assistance with problematic movies, tracks all movies, and automatically averages them)*
* A cell array of movie filenames
* A cell array of directory filenames
* A text file listing directories of movies

### Optional Arguments

| Argument | Description |
| :--- | :--- |
| `'scale', 'my_scale_movie.avi'` | Include if you have a second movie (`my_scale_movie.avi`) with an object of known length. |
| `'holepunch'` | Include if you drew a circle using a standard 6 mm diameter holepunch for calibration. |
| `'stimulus', 'my_stimulus.txt'` | Include if you have an optogenetics stimulus program file (`my_stimulus.txt`). |
| `'framerate', your_framerate_in_Hz` | Include if your framerate is **NOT 3 Hz**. |
| `'numworms', #worms` | Optional; include if number of worms is **< 20** or **> 30**. |
| `'no_collisions'` | Optional; include if you believe collisions between worms are unlikely. |

### Usage Examples
```matlab
% Track with a separate scale movie
MultiWormTracker('mymovie.avi', 'scale', 'my_scale_movie.avi');

% Track with a 6mm holepunch scale movie
MultiWormTracker('mymovie.avi', 'holepunch', 'scale', 'my_scale_holepunch_movie.avi');

% Analyze a movie recorded at 5 Hz with a scale movie
MultiWormTracker('mymovie.avi', 'scale', 'my_scale_movie.avi', 'framerate', 5);

% Analyze a movie with only 10 animals
MultiWormTracker('my_movie.avi', 'numworms', 10);

% Include an optogenetics stimulus program
MultiWormTracker('mymovie.avi', 'stimulus', 'my_stimulus.txt');
```

#### Example `my_stimulus.txt`
If you have an optogenetics stimulus program `my_stimulus.txt` that has 10 sec and 20 sec blue light pulses separated by 100 sec, format it as:
```text
off 100 sec
on 10 sec blue
off 100 sec
on 20 sec blue
off 100 sec
```

---

## Lawn Leaving Analysis

For experimental details, see *Bendesky et al. (2011)*. Draw a circle around a lawn using a standard **6 mm diameter holepunch** hole as a stencil.

```matlab
leaving_events_per_min_on_lawn = lawn_leaving_analysis(Moviename, framerate, timerbox_flag);
```

* Returns the mean lawn leaving events per worm-minute on the lawn in `leaving_events_per_min_on_lawn`.
* If your movie has a timerbox in the corner, set `timerbox_flag` to `1` or `'timerbox'`, else ignore it.
* If the program has a hard time finding the lawn edge or worms, the GUI will ask for your help.

```matlab
% Example: no timer, 1 frame/sec
leaving_events_per_min_on_lawn = lawn_leaving_analysis('mymovie.avi', 1)

% Example: timer box present, 1 frame/sec
leaving_events_per_min_on_lawn = lawn_leaving_analysis('mymovie.avi', 1, 'timerbox')
```

---

## Output Data Structures

### 1. Track Files (`rawTracks.mat`, `Tracks.mat`, `linkedTracks.mat`)
The data for each track are stored in `rawTracks.mat`, `Tracks.mat` (processed tracks), and `linkedTracks.mat` (track segments with closed gaps).

```matlab
myTrack_variable = load_Tracks('prefix.linkedTracks.mat')
```

`myTrack_variable` is a structure array of tracks, where each element is a single continuous track.

#### Fields in the `Track` Structure
* `Frames` — Actual frame from the start of the video
* `Time` — Time in seconds from the start of the video
* `Size` — Instantaneous area of the worm being tracked, in pixels
* `Eccentricity` — Instantaneous eccentricity of the ellipse that circumscribes the worm (`straight --> 1`, `circle --> 0`)
* `MajorAxes` — Major axis of the ellipse that circumscribes the worm
* `RingDistance` — Instantaneous distance of the animal from the copper ring (if it is there)
* `Image` — Binary image of the animal within the bounding box that circumscribes the worm
* `bound_box_corner` — `[x, y]` coordinates of the upper-left corner of the bounding box
* `body_contour` — Structure array with `[x, y]` coordinates of the animal's midline and curvature along the A–P axis
* `NumFrames` — Total number of frames in this track
* `numActiveFrames` — Number of frames in this track not censored due to ring-induced pirouettes
* `Height` — Height of the frame in pixels
* `Width` — Width of the frame in pixels
* `PixelSize` — Scale in mm/pixel
* `FrameRate` — Frame rate in Hz
* `Name` — Name of the movie file
* `SmoothX` — X-coordinates of track; smoothened with a sliding window of 1 sec
* `SmoothY` — Y-coordinates of track; smoothened with a sliding window of 1 sec
* `Direction` — Current heading of the animal's path (`0 deg` = up the screen)
* `Speed` — Instantaneous speed of the animal in mm/sec
* `original_track_indicies` — Indices of the original `rawTracks` that were linked
* `Wormlength` — Length of the animal in mm
* `AngSpeed` — Instantaneous angular speed of the animal in deg/sec
* `Reorientations` — Structure array describing each reorientation event (reversal, turn, omega, and combinations), reversal length, change in direction, eccentricity of the turn, etc.
* `State` — Instantaneous behavioral state code (`num_state_convert` can translate into English)
* `body_angle` — Instantaneous angle between the head, neck, and tail
* `head_angle` — Instantaneous angle between the head, neck, and midbody
* `tail_angle` — Instantaneous angle between the tail, lumbar point, and mid-body
* `midbody_angle` — Instantaneous angle between the neck, midbody, and lumbar point
* `curvature_vs_body_position_matrix` — Matrix for producing body curvature kymographs (rows are body segments `A --> P`, columns are frames)
* `Curvature` — Path curvature in deg/mm
* `mvt_init` — Indicates if a behavior was initiated at that frame
* `stimulus_vector` — Indicates whether a stimulus is present at that frame

#### Useful `Track_analysis` & `Track_tools` Functions
* `num_state_convert` — Convert a numerical behavior state code into English
* `find_Track` — Find `Tracks` and frames that meet the requested behavior criteria

---

### 2. Binned & Averaged Data (`BinData.mat`)
`BinData.mat` contains the binned and averaged data, and may be loaded with:
```matlab
my_binned_data_variable = load_BinData('prefix.BinData.mat')
```
The relevant `.txt` files contain the data from `BinData.mat` in a tab-delimited format which can be opened in Excel or with any text editor.

`my_binned_data_variable` is a structure which has arrays of behavioral parameters from tracks averaged in bins:
* **1-second bins (`time`, `n`):** Instantaneous values such as speed and fraction of animals in a given behavioral state are binned with a **1 sec** window. The times are in the `time` field, and the number of animals being averaged for each bin are in the `n` field.
* **10-second bins (`freqtime`, `n_freq`):** Event initiation frequencies are binned with a **10 sec** window. The times are in the `freqtime` field, and the number of animals being averaged for each bin are in the `n_freq` field.
* **Standard Deviation & Standard Error (`_s`, `_err`):** There are also fields for standard deviation (`_s`) and standard error (`_err`). For example:
  * `my_binned_data_variable.speed` — average speed for every second of the movie
  * `my_binned_data_variable.speed_s` — standard deviation of the speed for every second of the movie
  * `my_binned_data_variable.speed_err` — standard error of the speed for every second of the movie

#### Behavior Frequency & State Fraction Fields
The behavior fields in `BinData` all include fields for the frequency of initiating that behavior (`_freq`, 10 sec bin) and the fraction of animals currently in that behavioral state (`frac_`, 1 sec bin). For example, `sRevOmega` refers to short reversals coupled to an omega turn: `sRevOmega_freq` is the initiation frequency (10 sec bin) and `frac_sRevOmega` is the fraction of animals currently in a `sRevOmega` state (1 sec bin).

* `Rev` — All reversals
* `sRev` — All short reversals
* `lRev` — All long reversals
* `omegaUpsilon` — All turns
* `upsilon` — All upsilon turns
* `omega` — All omega turns
* `pure_Rev` — Reversals not immediately followed by a turn
* `pure_sRev` — Short reversals not immediately followed by a turn
* `pure_lRev` — Long reversals not immediately followed by a turn
* `pure_omegaUpsilon` — All turns not immediately preceded by a reversal
* `pure_upsilon` — Upsilon turns not immediately preceded by a reversal
* `pure_omega` — Omega turns not immediately preceded by a reversal
* `RevOmega` — Reversals immediately followed by an omega
* `RevOmegaUpsilon` — Reversals immediately followed by a turn
* `lRevOmega` — Long reversals immediately followed by an omega
* `lRevUpsilon` — Long reversals immediately followed by an upsilon
* `sRevOmega` — Short reversals immediately followed by an omega
* `sRevUpsilon` — Short reversals immediately followed by an upsilon
* `pause` — Stop moving

#### Instantaneous Behavior Parameters (1 sec bins)
* `speed` — Averaged over animals in a forward motion state in mm/sec
* `angspeed` — Angular speed in deg/sec
* `curv` — Track path curvature in deg/mm averaged over animals in a forward motion state
* `ecc` — Average eccentricity averaged over animals in a forward motion state
* `body_angle` — Average body angle in degrees averaged over animals in a forward motion state
* `head_angle` — Average head angle in degrees averaged over animals in a forward motion state
* `tail_angle` — Average tail angle in degrees averaged over animals in a forward motion state
* `revlength` — Average reversal length in terms of the length of the reversing animal's body
* `revlength_bodybends` — Average length of reversals in bodybends
* `revSpeed` — Average speed of reversing animals
* `delta_dir_rev` — Average change in direction of animals following a reversal
* `ecc_omegaupsilon` — Average minimal eccentricity of animals in a turn state
* `delta_dir_omegaupsilon` — Average change in direction of animals following a turn

#### State Transition Probabilities (1 sec bins)
There are five behavior states: **forward (`F`)**, **reverse (`R`)**, **pause (`P`)**, **omega turn (`O`)**, and **upsilon turn (`U`)**. Probabilities for transitioning from each state to each of the other states are calculated and binned every second. For example:
* `P_F_to_R` — Probability for an animal in a forward state transitioning to a reversal state

#### Useful `Bin_analysis` Functions
* `segment_statistics` — Extracts `BinData` values and statistics for a given field and time window
* `BinData_to_matrix` — Converts the requested fieldnames in `BinData` to a matrix (useful for PCA analysis)
* `alternate_binwidth_BinData` — Rebins the `BinData` with user-defined bin widths
* `extract_BinData` — Trims the `BinData` to a defined window of time

---

## Interactive Viewer (`WormPlayer`)

```matlab
WormPlayer('mymovie.linkedTracks.mat')
```
Launches a GUI viewer for playing individual worm tracks overlaid on the movie. Play around with it—you can zoom in and out. It may or may not be useful to you; it is admittedly a bit clunky.

---

## Citation & Contact

There are lots of other functions and programs hidden in the package... I need to document them all someday!

### Reference
For additional details and remarks, please see:
> Pokala and Flavell, *"Recording and quantifying C. elegans behavior"* in the ***C. elegans Methods and Applications*** volume of ***Methods in Molecular Biology***.

### Support & Contact
If you have any questions or problems, please let me know—I have found that anytime someone else uses my software, it fails in a novel manner! There are many additional functions for analyzing and displaying `Tracks` and binned data I haven't had a chance to properly document. Feel free to drop me an email to see whether a function to do what you want already exists or is a few edits away from existing:

* **Navin Pokala**  
  `navin (dot) pokala (at) nyit (dot) edu`
